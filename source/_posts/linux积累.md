---
title: linux 下载deb和其依赖
date: 2022-06-24 09:49:01
tag: 
- linux
- deb
- depends
categories:
- [linux]
---

# linux 下载deb和其依赖

1. 安装apt-rdepends

   `apt install apt-rdepends`

2. 下载依赖，比如vim

   `apt download $(apt-rdepends vim | grep -v "^ ")`

3. 如果出现错误比如：`E: Can't select candidate version from package debconf-2.0 as it has no candidate`可以使用一下命令：

   `apt-get download $(apt-rdepends vim | grep -v "^ " | sed 's/debconf-2.0/debconf/g')`

4. 拷贝deb文件到新的环境，安装之：`dpkg -i *`



<!-- more -->