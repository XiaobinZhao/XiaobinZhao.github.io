---
title: FastAPI
date: 2020-10-20 09:49:01
tags:
- FastAPI
categories:
- FastAPI
---

# 前言

学习了一下FastAPI，感觉其实现了目前python-web需要的大部分功能，可以说是这些python web框架的集大成者。我所关注的特性如下：
1. 自动化文档。文档标准是OpenAPI，也就是swagger。目前xView项目实现了与FastAPI类似的自动化文档，但是欠缺了一点是：没有使用Pydantic这样的声明类型。request参数的构造也咩有FastAPI灵活便捷。

<!-- more -->

2. 支持websock。目前xView项目需要单独开启socket服务

3. 支持ORM。支持集成sqlAlchemy+alembic

4. 支持plugin方式拦截请求。在FastAPI叫MiddleWare

5. 异步io

让我惊喜的是：
1. 可以同时支持FastAPI+Django[Flask/bottle等]并存
2. 可以支持root-url设置代理，这样文档生成就不会需要特别修改
3. 可以设置API CallBack 文档，这样我们的API文档就会非常友好
4. 集成很多MiddleWare，比如
  - SentryMiddleware，做日志重定向并管理
  - TimingMiddleware，统计每一个API请求耗时

# 基础知识
## type hints(类型提示)
- python3.0 引入 **函数注释** [PEP 3107](http://www.python.org/dev/peps/pep-3107).这提供了注释函数参数和返回值的标准化方法。除了可以在运行时使用 `__annotations__`属性对它们进行自省外，这些注释没有附加任何语义。目的是鼓励通过元类，装饰器或框架进行实验。
- Python3.5.0 开始支持 **类型提示**，见[PEP 484](https://docs.python.org/3.5/whatsnew/3.5.html#whatsnew-pep-484)。
python3.0支持函数注释，但是对注释的语义没有做定义。经验表明，大多数函数注释用法都是为函数参数和返回值提供**类型提示**。显而易见的是，如果标准库包含基本定义和用于类型注释的工具，则对Python用户将是有益的。它的主要作用**是方便开发，供IDE和各种开发工具使用**，对代码运行不产生影响，运行时会过滤类型信息。
```python
def add(x: int, y:int) -> int:
    return x+y
```
运行如下命令：
`print(add(1,2))` 和 `print(add(1,2))` 结果分别是：`3` 和 `helloworld`。
也就是说，python3.5版本的type hint就是一个文档标识作用，便于IDE编辑和代码维护，不能进行代码层面的自动校验。
- python3.6 开始支持**变量注释**语法，见 [PEP 484](https://www.python.org/dev/peps/pep-0484).与3.0/3.5不同之处是：添加了变量注释，之前版本是方法返回值注释和方法参数注释。
```python
primes: List[int] = []

captain: str  # Note: no initial value!

class Starship:
    stats: Dict[str, int] = {}
```
就像函数注释一样，Python解释器不会在变量注释中附加任何特殊含义，而仅将它们存储在__annotations__类或模块的属性中。
- 3.7 支持优化类型提示。因为引入类型提示带来了一些问题：
  + 批注只能使用当前范围内已经可用的名称，换句话说，它们不支持任何形式的正向引用；和
  + 注释源代码会对Python程序的启动时间产生不利影响。
所以，3.7版本进行优化，使用方式是：
```python
from __future__ import annotations  # 在模块中导入，Python 4.0中的默认设置
```
所以想要实现fastAPI的功能，需要支持python3.6+，如果需要优化类型提示需要python3.7+
想了解有关类型的更多信息，来自 [mypy 的"速查表"](https://mypy.readthedocs.io/en/latest/cheat_sheet_py3.html)是不错的资源。


# OpenAPI与FastAPI
## Request
Request参数校验主要包含三种情况：
1. 请求url中的path包含的参数，比如： `/users/{user_id}`这样的url,user_id就是path param; 
2. 而形如：`/users?name=aaa`这样的url, name是query param;
3. 以上2中情况多出现于GET请求，而PUT/PATCH/POST这样的请求会包含request body,body内的参数为就是第三种情况
### path param
```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/items/{item_id}")
async def read_item(item_id: int):
    return {"item_id": item_id}
```
代码如上。
1. 使用pydantic声明式命名
2. 数据转换，fastAPI 通过上面的类型声明,自动转换item_id为int型
3. 数据校验，如果item_id不是整形数字，比如字符串或者浮点型，都将报错
4. API 标注和自动生成的文档
### query param
```python
from fastapi import FastAPI

app = FastAPI()

fake_items_db = [{"item_name": "Foo"}, {"item_name": "Bar"}, {"item_name": "Baz"}]


@app.get("/items/")
async def read_item(skip: int = 0, limit: int = 10):
    return fake_items_db[skip : skip + limit]
```
代码如上，如果请求的url如：`http://127.0.0.1:8000/items/?skip=0&limit=10`,那么skip：对应的值为 0，limit：对应的值为 10。
和path param 一样，你可以得到：
- 数据"解析"
- 数据校验
- 自动生成文档

### path param + query param
你可以同时声明多个路径参数和查询参数，FastAPI 能够识别它们。
```
from typing import Optional

from fastapi import FastAPI

app = FastAPI()


@app.get("/users/{user_id}/items/{item_id}")
async def read_user_item(
    user_id: int, item_id: str, q: Optional[str] = None, short: bool = False
):
    item = {"item_id": item_id, "owner_id": user_id}
    if q:
        item.update({"q": q})
    if not short:
        item.update(
            {"description": "This is an amazing item that has a long description"}
        )
    return item
```
以上无论是 path param， 还是query param，还是2者并存，都可以设置参数为：
- 可选/必填
- 默认值
- 参数的一些额外校验
  - 字符串参数长度(最长/最短)，比如  `q: Optional[str] = Query(None, max_length=50)`
  - 字符串参数正则`q: Optional[str] = Query(None, regex="^fixedquery$")` 
  - 数字参数范围控制，比如：大于，小于，大于等于等
  - 查询参数是一个列表， 比如：`http://localhost:8000/items/?q=foo&q=bar`
- 设置更多参数，可以用为OpenAPI文档，比如：
  - title
  - description
- 别名参数
- 弃用设置

### Request body
需要先创建数据模型，这里的数据模型就是request的body的数据声明，
```
from typing import Optional

from fastapi import FastAPI
from pydantic import BaseModel


class Item(BaseModel):
    name: str
    description: Optional[str] = None
    price: float
    tax: Optional[float] = None

app = FastAPI()

@app.post("/items/")
async def create_item(item: Item):
    return item
```
按照以上方式，你将得到：
- 将请求体作为 JSON 读取。
- 转换为相应的类型（在需要时）。
- 校验数据。
  如果数据无效，将返回一条清晰易读的错误信息，指出不正确数据的确切位置和内容。
- 将接收的数据赋值到参数 item 中。
  由于你已经在函数中将它声明为 Item 类型，你还将获得对于所有属性及其类型的一切编辑器支持（代码补全等）。
- 为你的模型生成 JSON 模式 定义，你还可以在其他任何对你的项目有意义的地方使用它们。
- 这些模式将成为生成的 OpenAPI 模式的一部分，并且被自动化文档 UI 所使用。

当然你可以随意组合query parm、path param、request body.FastAPI能够正确的识别他们。

如果request body是一个复杂类型，比如：
```
{
    "item": {
        "name": "Foo",
        "description": "The pretender",
        "price": 42.0,
        "tax": 3.2
    },
    "user": {
        "username": "dave",
        "full_name": "Dave Grohl"
    }
}
```
你可以这么写：
```
from typing import Optional

from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    description: Optional[str] = None
    price: float
    tax: Optional[float] = None


class User(BaseModel):
    username: str
    full_name: Optional[str] = None


@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item, user: User):
    results = {"item_id": item_id, "item": item, "user": user}
    return results
```
在以上示例中，所有的属性或者变量的声明类型都是简单类型：int/str/bool/float, 还可以有复杂类型比如：List/Set/嵌套类型，或者直接使用Optional。pydantic还提供了一些特殊领域的声明，比如：HttpUrl。

## Request Filed
以上request path param/query param/requet  body其实都是Pydantic 的 Field的字段的实例。
```
from typing import Optional

from fastapi import Body, FastAPI
from pydantic import BaseModel, Field

app = FastAPI()


class Item(BaseModel):
    name: str
    description: Optional[str] = Field(
        None, title="The description of the item", max_length=300
    )
    price: float = Field(..., gt=0, description="The price must be greater than zero")
    tax: Optional[float] = None


@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item = Body(..., embed=True)):
    results = {"item_id": item_id, "item": item}
    return results
```
实际上，Query、Path 和其他你将在之后看到的类，创建的是由一个共同的 Params 类派生的子类的对象，该共同类本身又是 Pydantic 的 FieldInfo 类的子类。
Pydantic 的 Field 也会返回一个 FieldInfo 的实例。
Body 也直接返回 FieldInfo 的一个子类的对象。还有其他一些你之后会看到的类是 Body 类的子类。
请记住当你从 fastapi 导入 Query、Path 等对象时，他们实际上是返回特殊类的函数。

除了通过字段声明来控制一个属性的校验规则之外，还可以显式的指定。
```
class Item(BaseModel):
    name: str
    description: Optional[str] = None
    price: float
    tax: Optional[float] = None

    class Config:
        schema_extra = {
            "example": {
                "name": "Foo",
                "description": "A very nice Item",
                "price": 35.4,
                "tax": 3.2,
            }
        }
```
这里是增加了example属性，可以在openAPI 文档中看到。
## Response
与request不同的是，response的参数声明式在路径装饰器.
```
from typing import List, Optional

from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    description: Optional[str] = None
    price: float
    tax: Optional[float] = None
    tags: List[str] = []


@app.post("/items/", response_model=Item)
async def create_item(item: Item):
    return item
```
你将得到：
- 转换输出数据为声明类型。这里之所以把`response_model`声明放在路径装饰器上，是因为路径方法(比如上边示例的`create_item`)的返回值可能是字典/对象甚至没有返回值。使用装饰器来控制和序列化路径方法的返回值。
- 验证数据
- 响应api路径
- 自动文档

fastAPI可以自动转换SqlAlchemy ORM Model 到Schema Model。这样API就可以直接返回SqlAlchemy对象，例如：
```
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

class Item(ItemBase):
    id: int
    owner_id: int

    class Config:
        orm_mode = True

@app.post("/users/{user_id}/items/", response_model=Item)
def create_item_for_user(
    user_id: int, item: Item, db: Session = Depends(get_db)
):
    db_item = models.Item(**item.dict(), owner_id=user_id)
    db.add(db_item)
    db.commit()
    db.refresh(db_item)
    return db_item
```
上例子中， `db_item`是SqlAlchemy ORM对象，response_model是schema对象。这里会自动进行转化，前提是需要schema class 定义`orm_mode`为true

## Header
```
from typing import Optional

from fastapi import FastAPI, Header

app = FastAPI()


@app.get("/items/")
async def read_items(user_agent: Optional[str] = Header(None)):
    return {"User-Agent": user_agent}
```
需要注意的是：
- 不区分大小写
- header中的连字符`-`会自动换成下划线`_`
## Cookie等
```
from typing import Optional

from fastapi import Cookie, FastAPI

app = FastAPI()


@app.get("/items/")
async def read_items(ads_id: Optional[str] = Cookie(None)):
    return {"ads_id": ads_id}
```
# 依赖
FastAPi的Dependency类似于提取公共方法，然后被多处调用，原理跟装饰器差不多。比如你有如下需求：
1. 具有共享逻辑（一次又一次地使用相同的代码逻辑）。
2. 共享数据库连接。
3. 强制执行安全性，身份验证，角色要求等。
所有这些，同时最大程度地减少了代码重复。

依赖可以互相嵌套，即：子依赖

# 中间件
中间件（MiddleWare）可以拦截每一个request，处理请求前和response前的一些行为。
- 它接受应用程序中的每个请求。
- 然后，它可以对该请求执行某些操作或运行任何需要的代码。
- 然后，它传递要由应用程序其余部分（通过某些路径操作）处理的请求。
- 然后，它将获取应用程序生成的响应（通过某些路径操作）。
- 它可以对响应做出响应或运行任何需要的代码。
- 然后返回响应。
类似于xView bottle中的plugin。

# ORM
可以使用SqlAlchemy或者peewee(peewee是一个微型的ORM框架)。并且FastAPi支持自动转化SqlAlchemy对象和schema对象。
# 后台任务和Celery
如果返回response之后要运行一些操作，可以使用后台任务。
```
from fastapi import BackgroundTasks, FastAPI

app = FastAPI()


def write_notification(email: str, message=""):
    with open("log.txt", mode="w") as email_file:
        content = f"notification for {email}: {message}"
        email_file.write(content)


@app.post("/send-notification/{email}")
async def send_notification(email: str, background_tasks: BackgroundTasks):
    background_tasks.add_task(write_notification, email, message="some notification")
    return {"message": "Notification sent in the background"}
```
response也可以返回202，然后任务继续在后台执行。
与之类似的是Celery:一个分布式多任务处理队列。
# 静态资源
FastAPi也可以返回静态资源，比如：图片/文件。需要 单独安装`aiofiles`.
```
from fastapi import FastAPI
from fastapi.staticfiles import StaticFiles

app = FastAPI()

app.mount("/static", StaticFiles(directory="static"), name="static")
```
mount 是指在特定路径中添加完整的“独立”应用程序，然后负责处理所有子路径。
这与使用APIRouter完全独立的已安装应用程序不同。主应用程序中的OpenAPI和文档不会包含mount应用程序等中的任何内容。
# 子应用
如果需要两个独立的FastAPI应用程序，以及它们各自的独立OpenAPI和文档ui，则可以拥有一个主应用程序并“装载”一个（或多个）子应用程序。
```
from fastapi import FastAPI

app = FastAPI()


@app.get("/app")
def read_main():
    return {"message": "Hello World from main app"}


subapi = FastAPI()


@subapi.get("/sub")
def read_sub():
    return {"message": "Hello World from sub API"}


app.mount("/subapi", subapi)
```
子应用是以mount方式挂载到主应用中的。子应用与主应用区别之处就是文档是分开的，其他都一样。文档访问的时候，主应用默认地址： `http://127.0.0.1:8000/docs`, 子应用默认地址: `http://127.0.0.1:8000/subapi/docs`
# websocket
```
from fastapi import FastAPI, WebSocket
from fastapi.responses import HTMLResponse

app = FastAPI()

html = """
<!DOCTYPE html>
<html>
    <head>
        <title>Chat</title>
    </head>
    <body>
        <h1>WebSocket Chat</h1>
        <form action="" onsubmit="sendMessage(event)">
            <input type="text" id="messageText" autocomplete="off"/>
            <button>Send</button>
        </form>
        <ul id='messages'>
        </ul>
        <script>
            var ws = new WebSocket("ws://localhost:8000/ws");
            ws.onmessage = function(event) {
                var messages = document.getElementById('messages')
                var message = document.createElement('li')
                var content = document.createTextNode(event.data)
                message.appendChild(content)
                messages.appendChild(message)
            };
            function sendMessage(event) {
                var input = document.getElementById("messageText")
                ws.send(input.value)
                input.value = ''
                event.preventDefault()
            }
        </script>
    </body>
</html>
"""


@app.get("/")
async def get():
    return HTMLResponse(html)


@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()
    while True:
        data = await websocket.receive_text()
        await websocket.send_text(f"Message text was: {data}")
```

# 事件
您可以定义在应用程序启动之前或关闭应用程序时需要执行的事件处理程序（函数）。
```
from fastapi import FastAPI

app = FastAPI()

items = {}


@app.on_event("startup")
async def startup_event():
    items["foo"] = {"name": "Fighters"}
    items["bar"] = {"name": "Tenders"}


@app.on_event("shutdown")
def shutdown_event():
    with open("log.txt", mode="a") as log:
        log.write("Application shutdown")
        
        
@app.get("/items/{item_id}")
async def read_items(item_id: str):
    return items[item_id]
```
# 测试
fastAPI测试基于pytest. fastAPi继承了starlette的testClient, testClient又包装了requests.
```
from fastapi import FastAPI
from fastapi.testclient import TestClient

app = FastAPI()


@app.get("/")
async def read_main():
    return {"msg": "Hello World"}


client = TestClient(app)


def test_read_main():
    response = client.get("/")
    assert response.status_code == 200
    assert response.json() == {"msg": "Hello World"}
```
另外，testClient还可以用于：
- 测试websocket。
```
def test_websocket():
    client = TestClient(app)
    with client.websocket_connect("/ws") as websocket:
        data = websocket.receive_json()
        assert data == {"msg": "Hello WebSocket"}
```
- 测试事件：启动和关闭
- override 依赖
- 单独出测试数据库

# 模板
fastAPI没有提供template，如果要用需要可以使用：jinja2。
```
from fastapi import FastAPI, Request
from fastapi.responses import HTMLResponse
from fastapi.staticfiles import StaticFiles
from fastapi.templating import Jinja2Templates

app = FastAPI()

app.mount("/static", StaticFiles(directory="static"), name="static")


templates = Jinja2Templates(directory="templates")


@app.get("/items/{id}", response_class=HTMLResponse)
async def read_item(request: Request, id: str):
    return templates.TemplateResponse("item.html", {"request": request, "id": id})
```
# 部署/Server
一般情况下，fastAPi只作为应用的载体，需要一个server容器来部署应用，这个server我们建议使用uvicorn, 比如：
`uvicorn main:app --reload`
这样的命令行方式适合部署时使用，如果要进行debug,可以使用以下方式部署：
```
import uvicorn
from fastapi import FastAPI

app = FastAPI()


@app.get("/")
def root():
    a = "a"
    b = "b" + a
    return {"hello world": b}


if __name__ == "__main__":
    uvicorn.run(app, host="0.0.0.0", port=8000)
```
# 项目生成器
可以使用项目生成器来进行一键初始化：
GitHub：https : //github.com/tiangolo/full-stack-fastapi-postgresql

全栈FastAPI PostgreSQL-功能
1. 完全Docker集成（基于Docker）。
2. Docker Swarm模式部署。
3. Docker用于本地开发的集成和优化。
4. 使用Uvicorn和Gunicorn的可投入生产的 Python Web服务器。
5. Python FastAPI后端：
    - 快速：非常高的性能，看齐的NodeJS和围棋（感谢Starlette和Pydantic）。
    - 直观：强大的编辑器支持。完成无处不在。调试时间更少。
    - 简易：旨在易于使用和学习。减少阅读文档的时间。
    - 短：最小化代码重复。每个参数声明中的多个功能。
    - 健壮：获取可用于生产的代码。具有自动交互式文档。
    - 基于标准：基于（并完全兼容）API的开放标准：OpenAPI和JSON Schema。
    - 许多其他功能包括自动验证，序列化，交互式文档，使用OAuth2 JWT令牌进行身份验证等。
    - 默认情况下，安全密码哈希。
6. JWT令牌认证。
7. SQLAlchemy模型（独立于Flask扩展，因此可以直接与Celery工作者一起使用）。
8. 用户的基本启动模型（根据需要修改和删除）。
9. alembic迁移。
10. CORS（跨源资源共享）。
11. 可以选择性地从后端的其余部分导入和使用模型和代码的celery。
12. 基于Pytest的 REST后端测试，与Docker集成在一起，因此您可以独立于数据库测试完整的API交互。由于它在Docker中运行，因此每次都可以从头开始构建新的数据存储（因此您可以使用ElasticSearch，MongoDB，CouchDB或任何您想要的东西，只需测试API是否有效）。
13. 与Jupyter Kernels轻松实现Python集成，可通过 Atom Hydrogen或Visual Studio Code Jupyter之类的扩展来进行远程或Docker内部开发。
14. Vue前端：
    - 用Vue CLI生成。
    - JWT认证处理。
    - 登录视图。
    - 登录后，进入主仪表板视图。
    - 具有用户创建和版本的主仪表板。
    - 自我用户版。
    - Vuex。
    - Vue路由器。
    - Vuetify提供精美的材料设计组件。
    - TypeScript。
    - 基于Nginx的 Docker服务器（配置为与Vue-router完美配合）。
    - Docker多阶段构建，因此您无需保存或提交已编译的代码。
    - 前端测试在构建时运行（也可以禁用）。
    - 尽可能采用模块化设计，因此可以立即使用，但是您可以使用Vue - CLI重新生成或根据需要创建它，然后重新使用所需的内容。
15. PGAdmin for PostgreSQL数据库，您可以对其进行修改以轻松使用PHPMyAdmin和MySQL。
16. Flower为celery工作监视。
17. 使用Traefik可以在前端和后端之间实现负载平衡，因此您可以将两者都放在同一个域中，以路径分隔，但可以通过不同的容器进行服务。
18. Traefik集成，包括让我们加密HTTPS证书自动生成。
19. GitLab CI（连续集成），包括前端和后端测试。

## 项目生成器使用

按照生成器的说明步骤，输入适当参数即可得到一个 full-stack的项目，包含后端api、前端vue、docker 部署。

### backend: fastAPI 

backend 服务启动2个容器：python FastAPI应用和celery应用。

#### python FastAPI应用

1. 使用python3.7
2. 泛型和类型提示（type hints）
3. fastAPI官方出品docker容器：tiangolo/uvicorn-gunicorn-fastapi:python3.7
4. 使用Poetry来做包管理工具，代替Pipenv。
   1. python 与java/JavaScript等语言的project隔离是不一样的。python的项目依赖包都是统一安装到site-packages目录下，如果不同project依赖了不同版本的同一模块，那么后安装的会卸载掉先安装的。所以python需要为每一个项目进行单独隔离，所以virtualenv应运而生。
   2. 那么讨论python的依赖管理一般就指 依赖管理+虚拟环境。最初的工具就是pip+virtualenv，pip用来做包管理，virtualenv用来做虚拟环境。那么就带来问题：
      1. 需要同时使用2个工具
      2. 不能动态更新requirements.txt，这点尤其突出。这种文本格式的文件只能记录依赖包的名称，不能像yaml/json/xml一样记录更多的环境信息和参数。每次更新都是需要手动执行`pip freeze > requirements.txt`，如果那次遗漏，那么后患无穷。
   3. 因此，pipenv诞生了。
   4. pipenv可以看成是pip+virtualenv两款工具的合体，它集合了pip的依赖包管理和virtualenv虚拟环境 管理于一身。另外，在依赖包记录方面使用Pipfile替代原来的requirements.txt。而且，它能够自动记录并更新记录文件，这样就不在需要手动执行命令来更新requirements.txt。但是他依然有很多缺陷：
      1. Lock速度缓慢
      2. 强行更新不相干依赖
      3. 依赖处理效果较差。
   5. `当当~当~当~~~`！Poetry出现了
   6. poetry是一款可以管理Python依赖、环境，同時可以用于Python工程打包和发布的一款第三方工具包。poetry通过配置文件pyproject.toml来完成依赖管理、环境配置、基本信息配置等功能。相当于把Python項目中的Pipfile、setup.py、setup.cfg、requirements.txt、MANIFEST.in融合到一起。通过pyproject.toml文件，不仅可以配置依赖包，还可以用于区分开发、测试、生产环境、配置源路径。
5. 使用Tenacity做重试，判断DB是否就绪。[Tenacity](https://tenacity.readthedocs.io/)不兼容[retry](https://github.com/invl/retry)的api并且做了一些重要的功能和bug 修复。
   ```python
   import logging
   from tenacity import after_log, before_log, retry, stop_after_attempt, wait_fixed
   
   logging.basicConfig(level=logging.INFO)
   logger = logging.getLogger(__name__)
   
   max_tries = 60 * 5  # 5 minutes
   wait_seconds = 1
   
   @retry(
       stop=stop_after_attempt(max_tries),
       wait=wait_fixed(wait_seconds),
       before=before_log(logger, logging.INFO),
       after=after_log(logger, logging.WARN),
   )
   def init() -> None:
       try:
           # Try to create session to check if DB is awake
           db = SessionLocal()
           db.execute("SELECT 1")
       except Exception as e:
           logger.error(e)
           raise e
   ```
#### celery应用

1. 使用"uvicorn.workers.UvicornWorker" 
2. 容器启动之前使用Tenacity重试DB是否就绪

### frontend: vue
1. typescript
2. tslint
3. 测试使用vue-test-utils单元测试
4. Vue.js 的验证库VeeValidate
5. vuetify，是一个基于vue2.0，为移动而生的组件框架，一个渐进式的UI框架


