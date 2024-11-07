---
title: Hexo的一些配置
tags:
  - hexo
  - landscape
abbrlink: 7629139c
date: 2020-12-28 12:31:46
---

我的Blog使用的是默认主题，也就是 landscape。这里主要介绍了显示摘要，修改路径，配置证书等问题。


<!-- more -->


#### 1. 文章只显示摘要

hexo默认文章全文显示，很不方便，可以通过 <!-- more --> 语句实现阅读全文效果。

注意：
1. 在 <!-- more --> 语句前面，不能存在 markdown 语句；
2. 修改 node_modules/hexo-theme-landscape/_config.yml 文件中的 excerpt_link 值，可以自定义 Read More 文字。

#### 2. 给网站添加 favicon.ico

修改 node_modules/hexo-theme-landscape/_config.yml 配置文件。

```
favicon: favicon.ico
```

然后将 favicon.ico 文件放在 node_modules/hexo-theme-landscape/source/css/images 目录中。

#### 3. 添加备案信息

修改 themes/landscape/layout/_partial/footer.ejs 文件，添加备案信息。

```javascript
<footer id="footer">
  <% if (theme.sidebar === 'bottom'){ %>
    <%- partial('_partial/sidebar') %>
  <% } %>
  <div class="outer">
    <div id="footer-info" class="inner">
      &copy; <%= date(new Date(), 'YYYY') %> <%= config.author || config.title %><br>
      <%= __('powered_by') %> <a href="http://hexo.io/" target="_blank">Hexo</a>
      <div>
        <a href="https://beian.miit.gov.cn/" target="_blank">备案信息：xxxxx</a>
      </div>
    </div>
  </div>
</footer>
```

#### 4. 设置Blog的语言

以中文为例，修改 _config.yml 配置文件。

```
language: zh-CN
```
如果想要中英混搭，可以修改语言配置文件，文件为yml格式，放在node_modules/hexo-theme-landscape/languages 目录。

#### 5. URL路径不使用文件名

hexo的URL默认使用文件名作为路径，对于中文文件名，很不美观，可以使用 hexo-abbrlink 插件来生产唯一链接。

```shell
安装 hexo-abbrlink

npm install hexo-abbrlink --save

# 修改 修改 _config.yml
# 这里 paper 是路由，可以自定义

permalink: paper/:abbrlink.html
abbrlink:
  alg: crc32
  rep: hex
```

#### 6. 添加RSS订阅

安装 hexo-generator-feed

```
npm install hexo-generator-feed
```

在 _config.yml 中添加配置项

```
# RSS订阅
plugin:
  - hexo-generator-feed
# Feed Atom
feed:
  type: atom
path: atom.xml
limit: 10
```

在主题的 _config.yml 在添加菜单 rss: /atom.xml。


#### 7. 在文章中引用图片

在 source 目录下创建一个 images 文件夹，然后将图片放在此处，在 markdown 中，使用相对路径引用即可，比如：

```
[示例图片](/images/demp.png)
```

注意：是在 source 目录下，不是public，如果创建在public目录下，那么在需要清理数据，执行```hexo clean```命令的时候会被删除。

#### 8. apple-touch-icon

使用iPhone访问网站的时候，发现access.log报了几条日志。

```
404 GET /apple-touch-icon-120x120.png
404 GET /apple-touch-icon.png
404 GET /apple-touch-icon-120x120-precomposed.png
```

搜索了一下，原来是添加这个图片之后，在Safari浏览器的收藏夹里会显示大图标，ding到桌面之后也会显示一个图标，如果不设置，只会显示域名的首字母。

制作两张png图片，命名为 apple-touch-icon.png 和 apple-touch-icon-precomposed.png，放在source目录下即可。

#### 9. 添加数字证书

建议在 nginx 里面配置数字证书, 修改 /etc/nginx/nginx.conf文件，然后将443端口的证书配置注释去掉，再将证书文件拷贝到相应目录，然后执行 nginx -s reload 即可。

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
更多 nginx 的配置可以点击[这里](https://mastersword.cn/paper/5daddb41.html)，查看另一篇文章。


*参考*
https://hexo.io/zh-cn/docs/configuration.html
