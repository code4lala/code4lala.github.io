---
layout: post
comments: true
title:  "从OBS到php，搭建自己一个人的直播环境"
date:   2019-11-02 22:29:52 +0800
tags: 备忘 ubuntu nginx rtmp 直播 live php html
lang: zh
---

# 准备
1. 推流机器，跑`Open Broadcast Software`
2. 有公网ip的机器，跑`nginx`，跑`rtmp`服务
3. 看直播的机器，跑`浏览器`

# 服务端
## 解除防火墙端口1935限制
脚本复制的别人的改的，参考链接
- [https://www.91yun.co/archives/1606](https://www.91yun.co/archives/1606)
- [https://www.dwhd.org/20150915_162703.html](https://www.dwhd.org/20150915_162703.html)
- [https://gtour.info/how-to-configure-iptables-firewall-in-ubuntu-16-04/](https://gtour.info/how-to-configure-iptables-firewall-in-ubuntu-16-04/)
- [https://ruiruigeblog.com/2018/05/08/iptables防止shadowsocks用户发送垃圾邮件/](https://ruiruigeblog.com/2018/05/08/iptables%E9%98%B2%E6%AD%A2shadowsocks%E7%94%A8%E6%88%B7%E5%8F%91%E9%80%81%E5%9E%83%E5%9C%BE%E9%82%AE%E4%BB%B6/)
- [https://blog.hackroad.com/operations-engineer/linux_server/12880.html](https://blog.hackroad.com/operations-engineer/linux_server/12880.html)
- [https://www.v2ex.com/t/136612](https://www.v2ex.com/t/136612)

``` bash
#!/bin/bash
iptables -L -n 2>&1 | tee -a "_.iptables.log"
iptables -F        #清除预设表filter中的所有规则链的规则
iptables -X        #清除预设表filter中使用者自定链中的规则
iptables -Z        #计数器清零

iptables -P INPUT DROP
iptables -P OUTPUT ACCEPT
iptables -P FORWARD ACCEPT

#ping
iptables -A INPUT -p icmp --icmp-type echo-request -j ACCEPT
iptables -A OUTPUT -p icmp --icmp-type echo-reply -j ACCEPT
#双向
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

#SSH
iptables -A INPUT -p tcp --dport 26103 -j ACCEPT
#www 80
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
#https 443
iptables -A INPUT -p tcp --dport 443 -j ACCEPT
#rtmp
iptables -A INPUT -p tcp --dport 1935 -j ACCEPT

#对于OUTPUT规则，因为预设的是ACCEPT，所以要添加DROP规则，减少不安全的端口链接。
iptables -A OUTPUT -p tcp -m multiport --dport 25,366,465 -j DROP
iptables -A OUTPUT -p udp -m multiport --dport 25,366,465 -j DROP
iptables -A OUTPUT -p udplite -m multiport --dport 25,366,465 -j DROP
iptables -A OUTPUT -p sctp -m multiport --dport 25,366,465 -j DROP
iptables -A OUTPUT -p dccp -m multiport --dport 25,366,465 -j DROP

iptables -A OUTPUT -p tcp -m multiport --sport 25,366,465 -j DROP
iptables -A OUTPUT -p udp -m multiport --sport 25,366,465 -j DROP
iptables -A OUTPUT -p udplite -m multiport --sport 25,366,465 -j DROP
iptables -A OUTPUT -p sctp -m multiport --sport 25,366,465 -j DROP
iptables -A OUTPUT -p dccp -m multiport --sport 25,366,465 -j DROP

#FORWARD规则
iptables -A FORWARD -p tcp -m multiport --dport 25,366,465 -j DROP
iptables -A FORWARD -p udp -m multiport --dport 25,366,465 -j DROP
iptables -A FORWARD -p udplite -m multiport --dport 25,366,465 -j DROP
iptables -A FORWARD -p sctp -m multiport --dport 25,366,465 -j DROP
iptables -A FORWARD -p dccp -m multiport --dport 25,366,465 -j DROP

iptables -A FORWARD -p tcp -m multiport --sport 25,366,465 -j DROP
iptables -A FORWARD -p udp -m multiport --sport 25,366,465 -j DROP
iptables -A FORWARD -p udplite -m multiport --sport 25,366,465 -j DROP
iptables -A FORWARD -p sctp -m multiport --sport 25,366,465 -j DROP
iptables -A FORWARD -p dccp -m multiport --sport 25,366,465 -j DROP

#最后规则拒绝所有不符合以上所有的
iptables -A INPUT -j DROP

echo "iptables ok";

iptables -L -n
cat /etc/network/interfaces
```
## 安装nginx和rtmp模块
参考教程
- [https://opensource.com/article/19/1/basic-live-video-streaming-server](https://opensource.com/article/19/1/basic-live-video-streaming-server)
## 开启rtmp模块 配置nginx.conf
路径`/etc/nginx/nginx.conf`
``` bash
http {
    ... # rtmp需要和http同级
}

rtmp {
  server {
    listen 1935;
    chunk_size 4096;

    application stream {
      live on; # 开启串流
      record off; # 关闭录像
      # 身份认证
      on_publish http://127.0.0.1/rtmp_auth.php;
      # HTTP Live Streaming
	  hls on;
	  # 下边这行制定了输出的.m3u8的路径，后边会用到
      hls_path /var/www/html/live.code4lala.vip/hls;
      hls_fragment 5s;
      hls_playlist_length 60s;
      hls_cleanup on;
      hls_continuous on;
    }
  }
}
```
完事重启`nginx`，命令
``` bash
sudo service nginx restart|reload|status
```
如果配置文件有误，执行status查看状态能看到哪个配置文件的哪一行有误。

上边加了一个身份认证，所以要配置身份认证的服务器(前提需要保证php脚本能跑)

## 配置127网站和直播网站
配置文件`/etc/nginx/sites-available/default`，我所有的网站全都放到这一个配置文件里边了，所以就改这一个就行。
``` bash
# Default server configuration
#
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    root /var/www/html/default;
    index index.php;

    # 这个默认是 server_name _
    # 需要修改成自己的公网ip
    server_name 162.219.124.246;

    ... # 省略没什么影响的篇幅
}

# 127.0.0.1
server {
    listen 80; # 注意这个后边不能加 default_server
    root /var/www/html/127;
    index index.php;
    server_name 127.0.0.1;

    location / {
        # First attempt to serve request as file, then
        # as directory, then fall back to displaying a 404.
        try_files $uri $uri/ =404;
    }

    # pass PHP scripts to FastCGI server
    #
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php7.2-fpm.sock;
    }
}

# http 301 跳转到 https
# 这段是 certbot 自动加上的
server {
    if ($host = live.code4lala.vip) {
        return 301 https://$host$request_uri;
    } # managed by Certbot

    listen 80;
    listen [::]:80;

    server_name live.code4lala.vip;
    return 404; # managed by Certbot
}

# live.code4lala.vip
server {
    listen 443 ssl ;
    listen [::]:443 ssl ;

    server_name live.code4lala.vip;

    root /var/www/html/live.code4lala.vip;
    index index.html index.php;
    types {
        application/vnd.apple.mpegurl m3u8;
        video/mp2t ts;
    }
    add_header Cache-Control no-cache;

    location / {
        # First attempt to serve request as file, then
        # as directory, then fall back to displaying a 404.
        try_files $uri $uri/ =404;
    }

	# 这个private文件夹放mysql密码什么的，禁止外部访问，但是php代码能访问。预留给后边用
    location /private/ {
        return 404;
    }

    # pass PHP scripts to FastCGI server
    #
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php7.2-fpm.sock;
    }
    ssl_certificate /etc/letsencrypt/live/cloud.code4lala.vip/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/cloud.code4lala.vip/privkey.pem; # managed by Certbot
}
```
## 127网站php认证脚本
接着填充127网站的`rtmp_auth.php`。参考教程
- [https://smartshitter.com/musings/2018/06/nginx-rtmp-streaming-with-slightly-improved-authentication/](https://smartshitter.com/musings/2018/06/nginx-rtmp-streaming-with-slightly-improved-authentication/)
``` bash
<?php
# 推流地址
# rtmp://live.code4lala.vip/stream/
# ?name=lala&psk=your_password_used_in_obs

$username = $_POST["name"];
$password = $_POST["psk"];
$valid_users = array("lala" => "your_password_used_in_obs");
if ($username == "" | $password == "") {
  http_response_code(404); # return 404 "Not Found"
} else if ($valid_users[$username] == $password) {
  http_response_code(201); # return 201 "Created"
} else {
  http_response_code(404); # return 404 "Not Found"
}
?>
```
注意这个php脚本虽然是在127网站，仅本机可访问，但它还是通过rtmp暴露在了外部，而且这个php脚本没有任何的安全防护，后续会做一些安全措施，请参见[https://github.com/code4lala/live.code4lala.vip](https://github.com/code4lala/live.code4lala.vip)，写文章时安全措施还没搞完。

## 直播主站html
参考链接：
- [https://candinya.com/posts/搭建自己的RTMP-HLS直播服务器/](https://candinya.com/posts/%E6%90%AD%E5%BB%BA%E8%87%AA%E5%B7%B1%E7%9A%84RTMP-HLS%E7%9B%B4%E6%92%AD%E6%9C%8D%E5%8A%A1%E5%99%A8/)

``` html
<!DOCTYPE HTML>
<html lang="zh-cn">
<head>
    <meta charset="UTF-8">
    <!--    dplayer-->
    <link class="dplayer-css" rel="stylesheet" href="https://cdn.jsdelivr.net/npm/dplayer/dist/DPlayer.min.css">
    <script src="https://cdn.jsdelivr.net/npm/hls.js/dist/hls.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/dplayer/dist/DPlayer.min.js"></script>
    <title>欢乐白给直播间</title>
</head>
<body>
<div id="dplayer"></div>
<script>
    const dp = new DPlayer({
        container: document.getElementById('dplayer'),
        live: true,
        video: {
			<!-- 这里就是流地址了 -->
            url: 'hls/.m3u8',
            type: 'hls'
        }
    });
</script>
</body>
</html>
```

# 客户端
OBS的推流地址，那个域名用ip也完全可以，毕竟也没有真的对应这个网站
``` bash
rtmp://live.code4lala.vip/stream/
?name=lala&psk=your_password_used_in_obs
```
看直播就打开网站就可以喽