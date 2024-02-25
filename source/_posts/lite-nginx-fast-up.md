---
title: 我的轻量nginx快速启动方案
date: 2024-02-06 16:04:50
tags:
---

## 我的需求

- 基于docker，但要极致的小
- 统管80/443端口
- 指定cf的15年的证书，certbot就不必了
- 使用最新nginx版本，开启http2

## 我的成果

- docker compose文件；
```bash
cat >docker-compose.yml<<EOF
version: '3'
    
services:
  nginx:
    image: nginx:1.25.3-alpine3.18-slim
    restart: unless-stopped
    network_mode: "host"
    volumes:
      - ./nginx_secrets:/etc/letsencrypt
      - ./user_conf.d:/etc/nginx/conf.d
EOF
```
- 证书目录和配置目录；
```bash
mkdir -p ngx/nginx_secrets ngx/user_conf.d
```
- 配置文件样例；
```bash
cat >user_conf.d/www1.conf<<EOF
server {
    listen 443 ssl reuseport http2;
    # 注意这里，reuseport，只有www1能用，www2要去掉；
    # 如需default_server，放最后，不能和reuseport共存；
    
    server_name aaa1.example.com;
    server_name aaa2.example.com;
    server_name aaa3.example.com;
    
    # Load the certificate files.
    ssl_certificate         /etc/letsencrypt/live/www1/fullchain.pem;
    ssl_certificate_key     /etc/letsencrypt/live/www1/privkey.pem;
    ssl_trusted_certificate /etc/letsencrypt/live/www1/chain.pem;
    
    # Load the Diffie-Hellman parameter.
    ssl_dhparam /etc/letsencrypt/dhparams/dhparam.pem;
    
    return 200 'Let\'s Encrypt certificate successfully installed!';
    add_header Content-Type text/plain;
}
EOF
```
- 证书目录，文件如上，不赘述；
- `nginx:1.25.3-alpine3.18-slim` 大小 5.06 MB

---

### 顺便来个`proxy_pass`的小测试

- 方案A
```
location /_a {
    proxy_pass http://localhost:6006;
}
```

- A结果
```
request: "GET /_a/ HTTP/2.0", upstream: "http://127.0.0.1:6006/_a/",
request: "GET /_a HTTP/2.0", upstream: "http://127.0.0.1:6006/_a"
request: "GET /_a/x HTTP/2.0", upstream: "http://127.0.0.1:6006/_a/x"
```

- 方案B
```
location /_a {
    proxy_pass http://localhost:6006/;   ==> 区别于A：这里多了个斜杠
}
```

- B结果
```
request: "GET /_a HTTP/2.0", upstream: "http://127.0.0.1:6006/"
request: "GET /_a/ HTTP/2.0", upstream: "http://127.0.0.1:6006//"
request: "GET /_a/x HTTP/2.0", upstream: "http://127.0.0.1:6006//x"
```

- 方案B2（推荐）
```
location /_a/ {    ==> 区别于B：所以最好这里的斜杠能对应
    proxy_pass http://localhost:6006/;   ==> 同B，所以这个叫B2
}
```

- B2结果
```
https://xx.xx.xx/_a  会自己重定向到   /_a/
request: "GET /_a/ HTTP/2.0", upstream: "http://127.0.0.1:6006/"
request: "GET /_a/x HTTP/2.0", upstream: "http://127.0.0.1:6006/x"
```

