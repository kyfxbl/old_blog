title: nginx简单配置
date: 2014-05-15 19:55
categories: web
---
nginx一些常见的配置
<!--more-->

# 自动跳转

希望实现的效果是，用户只要访问域名，自动跳转到index.html页面

原本配置为：

```
location / {
    root   /users/apple/git_local/YAE/YAE/frontend;
    index  /portal/nail/index.html;
}
```

这样浏览器里的URL还是www.xxx.com。如果页面上有链接使用相对路径，就会发生404错误，所以需要配置为：

```
rewrite ^/(index.html)?$ portal/nail/index.html redirect;

location / {
    root   /users/apple/git_local/YAE/YAE/frontend;
    index  /portal/nail/index.html;
}
```

浏览器的URL会变成www.xxx.com/portal/nail/index.html，这样相对路径就能正常访问了

# root和alias区别

请求资源的URL：

```
http://127.0.0.1/storeadmin/css/jquery.Jcrop.css
```

实际在机器上的地址：

/users/apple/git_local/YAE/src/storeadmin/static/css/jquery.Jcrop.css

一开始nginx配置成：

```
location /storeadmin {
    root    /users/apple/git_local/YAE/src/storeadmin/static;
}
```
结果404错误，错误日志信息：

open() "/users/apple/git_local/YAE/src/storeadmin/static/storeadmin/css/jquery.Jcrop.css" failed (2: No such file or directory)

需要改为alias：

```
location /storeadmin {
    alias    /users/apple/git_local/YAE/src/storeadmin/static;
}
```

区别在于，在location后面配置的路径，在root里不会被丢弃，而在alias会丢弃掉