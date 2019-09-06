---
layout: post
comments: true
title:  "ubuntu 12 linux deploy qt"
date:   2019-09-02 15:22:39 +0800
tags: 原创 qt linux ubuntu
lang: zh
---

`原创文章`

linux发布qt应用简略教程

1. 新版win10和旧版vmware会不兼容，互传文件时老是卡死，最新版win10装最新版vmware
2. 虚拟机安装Ubuntu12
3. 更换[清华像源](https://mirror.tun镜a.tsinghua.edu.cn/help/ubuntu/)
4. 安装新版本gcc和g++来支持C++11(我装的4.8版本，[示例教程](https://remyaraj89.wordpress.com/2014/08/05/c11-compiler-support-in-ubuntu-12-04/))
5. ln -s \<source\> \<destinatin\> 链接到新的gcc和g++
6. 安装cmake
7. 安装qt`./qt-opensource-linux-x64-5.6.3.run`
8. 装完之后qt在`<qt文件夹>\Tools\QtCreator\bin`中
9. 下载[linuxdeployqt](https://github.com/probonopd/linuxdeployqt/releases)
10. 安装新版`libc.so`下载deb包并安装[教程](https://stackoverflow.com/questions/19471683/lib-libc-so-6-version-glibc-2-17-not-found/33173649)
11. 现在linuxdeployqt就可以运行了