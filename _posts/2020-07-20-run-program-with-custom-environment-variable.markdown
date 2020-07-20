---
layout: post
comments: true
title:  "自定义环境变量启动软件"
date:   2020-07-20 14:37:48 +0800
tags: batch 环境变量
lang: zh
---

`原创`

钉钉运行老是掉线，图片发不出去，想到可能是代理问题就用`procexp`查看钉钉的 TCP/IP 连接检查了一下，发现钉钉运行时不管代理设置是啥样的，一定非要用系统环境变量中的`HTTP_PROXY`进行连接。
![钉钉代理配置.png](https://s1.ax1x.com/2020/07/20/UhDH29.png)
![钉钉连接本地代理.png](https://s1.ax1x.com/2020/07/20/UhroeP.png)

那么解决办法就是不让钉钉看到这个环境变量就可以了，一行bat解决。

``` bat
cmd /c "set HTTP_PROXY= & set HTTPS_PROXY= & set HTTP_PROXY & set HTTPS_PROXY & "C:\Users\Fengl\source\repos\Project1\Release\a a\Project1.exe"
```

为了验证上边那行bat能不能用，写了一段c代码：

``` cpp
#define _CRT_SECURE_NO_WARNINGS
#include <stdio.h>
#include <windows.h>

int main(int argc, char** argv)
{
	system("cmd /c set http_proxy");
	for (int i = 0; i < argc; i++)
	{
		printf("%s\n", argv[i]);
	}
	if (argc <= 1)return 0;
	char cmd[1024];
	char* p = cmd;
	strcpy(p, argv[0]);
	p += strlen(argv[0]);
	for (int i = 1; i < argc - 1; i++)
	{
		strcpy(p, " ");
		p += 1;
		strcpy(p, argv[i]);
		p += strlen(argv[i]);
	}
	system(cmd);
	return 0;
}
```

很疑惑的是，上边这行batch是正常运行的而且真的让钉钉读不出来HTTP_PROXY环境变量的值了。

又经过几次验证，发现后边加上一个或更多的双引号都可以正常运行，但是至少要一个双引号。

并没有明白batch中双引号的用法，待查明后补充。