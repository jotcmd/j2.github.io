---
title: nginx proxy_pass 的小测试
date: 2024-02-25 21:38:50
tags:
---

> 论`/`的重要性。。

- 方案A （`推荐`）
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

- 方案B2（`推荐`）
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
