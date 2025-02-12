# 原因分析

`unexpected token '<'` → JS 语法错误。

即浏览器将 html 作为 js 文件直接执行了，导致的报错。

那么为什么会将 html 作为 js 文件执行呢 🤔

# 真実はいつもひとつ 👮🏻

浏览器在请求某个文件的时候没有找到，服务器返回了 index.html 文件，浏览器直接执行了该文件导致报错。

那么为什么找不到且返回 html 文件呢 🤔

- 浏览器、服务器缓存问题

- nginx 配置中 try_files 优先级问题

```TypeScript
location /wf-proxy {
    alias     /app/web-wf;
    try_files $uri $uri/ /index.html;
    if ($request_filename ~ .*\.html$) {
        add_header Cache-Control no-cache;
    }
    expires 5m;
}
```

# 排查

1. 直接访问 xxxx/xxxx.js 查看文件是否存在。

2. 清除浏览器缓存

