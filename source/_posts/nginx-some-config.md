---
title: Nginx的一些配置
tags:
  - nginx
date: 2021-1-11 00:11:46
abbrlink: 5daddb41
---

网站启用https，就需要配置数字证书。不想被探测器扫描，就禁止通过IP访问。


<!-- more -->

#### 1. 安装，启动，停止，加载

以 centos7 为例
```
# 安装
yum install nginx

# 启动 
nginx

# 停止
nginx -s stop

# 加载配置文件
nginx -s reload
```

#### 2. 修改默认显示内容

nginx默认的首页，404，500等页面的显示内容太多了，容易暴露服务器信息，可以将 index.html, 404x.html, 500x.html 替换成自定义的文件，默认目录为 /usr/share/nginx/html。

#### 3. 添加数字证书

修改 /etc/nginx/nginx.conf文件，然后将443端口的证书配置注释去掉，再将证书文件拷贝到相应目录，然后执行 nginx -s reload 即可。

```
listen       443 ssl http2 default_server;
listen       [::]:443 ssl http2 default_server;
server_name  _;
root         /usr/share/nginx/html;

ssl_certificate "/etc/pki/nginx/server.crt";
ssl_certificate_key "/etc/pki/nginx/private/server.key";
ssl_session_cache shared:SSL:1m;
ssl_session_timeout  10m;
ssl_ciphers HIGH:!aNULL:!MD5;
ssl_prefer_server_ciphers on;

# Load configuration files for the default server block.
include /etc/nginx/default.d/*.conf;

location / {
proxy_pass http://localhost:4000;
}
```

#### 4. 禁止通过ip直接访问网站

如果不想服务器被扫描器探测，可以禁止通过IP访问网站。
修改 nginx.conf 文件，添加一个 server，注意不是修改原 server。

新增一个server配置，这里将 80 和 443 写在了一起，也可以分开写。

```
# forbidden ip:port access
server {
    listen 80 default; 
    listen 443 ssl http2 default_server;
    ssl_certificate "/etc/pki/nginx/server.crt";
    ssl_certificate_key "/etc/pki/nginx/private/server.key";
    ssl_session_cache shared:SSL:1m;
    ssl_session_timeout  10m;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;
    server_name _;
    return 403;
}
```

修改原server，将 server_name _ 更改为 server_name yourdomain.com;

```
# 修改原server
server {
    listen       443; # 只保留443, 去掉 ssl http2 default_server;
    listen       [::]:443 ssl http2 default_server;
    server_name  yourdomain.com;
```

然后执行 nginx -s reload 重新加载配置即可。

#### 5. http跳转到https

鉴于http越来越不安全，可以添加http跳转，在用户通过http访问的时候，自动跳转到 https。

这里采用 return 301 重定来实现。

```
server {
    listen 80;
    server_name yourdomain.com;
    return 301 https://$server_name$request_uri;
}
```

网络上还有 rewrite 和 error_page 的方法，rewrite已不推荐，而 error_page 方法我测试并未生效，详情可参考[这篇文章](https://cloud.tencent.com/developer/article/1504193)。

#### 6. no-www与www

是用户访问网站的时候，有 www 和没有 www 都显示一样的效果。

因为我的域名比较长，这里以有 www 跳转到 无 www 为例。

在配置中新增一个server，可以把80和443配置在一起，当然也可以分开，更详细内容可参考[这篇文章](https://www.jianshu.com/p/cec753473ec9)。

```
# www.example.com redirect example.com
server {
    listen 80;
    listen 443 ssl;
    ssl_certificate "/etc/pki/nginx/server.crt";
    ssl_certificate_key "/etc/pki/nginx/private/server.key";
    ssl_session_cache shared:SSL:1m;
    ssl_session_timeout  10m;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;
    server_name www.yourdomain.com;
    return 301 https://yourdomain.com$request_uri;
}

```

#### 7. 去掉 nginx 的版本号

在响应数据的header中会显示nginx的版本号，如果不想显示，可以在配置文件中添加 server_tokens 配置。

```
# /etc/nginx/nginx.conf

http {
        server_tokens off;
```

加在 http 下面会全部隐藏，如果你想80端口隐藏，443端口不隐藏，那么把这个配置添加在 server 下面。

#### 8. 调整日志格式

修改 nginx.conf 中的 log_format，即可对 nginx 日志进行格式调整。

默认格式

```json
log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

access_log  /var/log/nginx/access.log  main;
```

调整后

```json
log_format  main  '{"time": "$time_local", "status": $status, "remote_addr": "$remote_addr", '
                  '"request": "$request", "body_bytes_sent": $body_bytes_sent, '
                  '"request_time": $request_time, "http_referer": "$http_referer", '
                  '"user_agent": "$http_user_agent"}';

access_log  /var/log/nginx/access.log  main;
```

有些不可见字符，在 json 里面显示不友好，可以再调整一下

```json
log_format  main  '$time_local | $status | $remote_addr | "$request" | '
                      '"$http_referer" | "$http_user_agent" | '
                      '$body_bytes_sent | $request_time';
```

