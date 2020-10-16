---
title: hexo-Next + Github Page制作个人网站使用记录
description: 记录使用github page发布个人网站的方法和调整hexo-Next主题的步骤
tag: 
- hexo
- Next
categories:
- [hexo]
---
# hexo-Next 主题使用记录

1. 修改内容自动重载并启动server

   安装 `hexo-browsersync`即可：`npm install hexo-browsersync`

   安装成功之后，只需要正常执行 `hexo server` 或 `hexo s` 就能查看效果了。

   经过测试，当保存文章的 Markdown 文件时，网页会自动刷新。另外，修改主题配置文件并保存之后，网页也会自动刷新，非常 nice。唯一一个缺憾是修改站点配置文件没办法触发网页的自动刷新，需要重新执行 `hexo g`。

<!-- more -->

2. 背景动画
```yaml
# JavaScript 3D library.
# Dependencies: https://github.com/theme-next/theme-next-three
three:
  enable: fasle
  three_waves: true
  canvas_lines: false
  canvas_sphere: false
  
vendors:
  three: //cdn.jsdelivr.net/gh/theme-next/theme-next-three@1/three.min.js
  three_waves: //cdn.jsdelivr.net/gh/theme-next/theme-next-three@1/three-waves.min.js
  canvas_lines: //cdn.jsdelivr.net/gh/theme-next/theme-next-three@1/canvas_lines.min.js
  canvas_sphere: //cdn.jsdelivr.net/gh/theme-next/theme-next-three@1/canvas_sphere.min.js	
```

3. 