# websocket转发

```JavaScript
new WebSocket('ws://1.1.0.0:8083/ws')
```

```Shell
#必须添加的
map $http_upgrade $connection_upgrade {
     default upgrade;
     '' close;
}

 server {
     #普通的端口
     listen 8083;
     root /web_cloud/dist;

     location /{
             try_files $uri $uri/ /index.html;
     }
     # 正常的接口请求
     location /api/ {
             proxy_pass http://localhost:2021;
             proxy_set_header  Host $http_host;
             proxy_set_header  X-Real-IP  $remote_addr;
             proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;
             proxy_set_header  X-Forwarded-Proto $scheme;
     }
     # 转发ws
     location ^~ /ws {
             # 后台准备的websocket地址端口
             proxy_pass http://localhost:9092;
             # 其他参数都一样
             proxy_read_timeout 300s;
             proxy_send_timeout 300s;
             proxy_set_header  Host $http_host;
             proxy_set_header  X-Real-IP  $remote_addr;
             proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;
             proxy_set_header  X-Forwarded-Proto $scheme;
             proxy_http_version 1.1;
             proxy_set_header Upgrade $http_upgrade;
             proxy_set_header Connection $connection_upgrade;
     }
}
```

1. $http_upgrade是一个内置变量，表示客户端请求的Upgrade头部字段的值。这个字段通常用于指示客户端希望升级到的协议，例如从HTTP升级到WebSocket。

2. $connection_upgrade是一个自定义变量，通过map指令来设置它的值。

# alise部署前端

1. try_files：需要加上alise前缀

2. index.html中路径需要调整为 `/path/`

```Shell
location /path {
   alias "/usr/local/nginx/hsor_pad/";
   try_files $uri $uri/ /path/index.html;
   index  index.html index.htm;
}
```

