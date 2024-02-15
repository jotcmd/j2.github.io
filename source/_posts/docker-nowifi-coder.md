---
title: docker使用code server但不让它联网
date: 2024-02-13 22:04:50
tags:
---

## 我的需求

- docker启动code server，作为远程的在线编程环境；
- 但不让code server联网，倒不是担心MS的开源项目会怎么，主要是洁癖吧；
- 甚至不让code server的ip、https服务在公网暴露，这个是个小trick；

## 我的成果

1. 创建一个不能联网的docker网络

```bash
docker compose stop && docker compose rm -f 
docker network rm nowifi
docker network create --subnet=172.31.0.0/24 nowifi # 不能乱来、172.31估计时最大的段了
sudo iptables -I FORWARD -s 172.31.0.0/24 -j DROP
sudo iptables -I FORWARD -s 172.31.0.0/24 -m conntrack --ctstate ESTABLISHED,RELATED 

docker network ls # 用于查看
sudo iptables-save | grep 172.31 # 用于查看
# 如果需调试、用于取消设置
sudo iptables -D FORWARD -s 172.31.0.0/24 -j DROP
sudo iptables -D FORWARD -s 172.31.0.0/24 -m conntrack --ctstate ESTABLISHED,RELATED
```

2. 启动code server，并使用上面的网络

```bash
cat >docker-compose.yml<<EOF
version: '3'
services:
  cs1:
    container_name: cs1
    image: lscr.io/linuxserver/code-server:latest
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Taipei
      - DEFAULT_WORKSPACE=/root
      - HASHED_PASSWORD=xxx
    #ports:  # 甚至都不开localhost映射，确实也没必要
    #  - 127.0.0.1:18080:8443
    restart: unless-stopped
    volumes:
      - ./config:/config
      - \${HOME}/dev:/root
    network_mode: nowifi  # 使用上面的网络
EOF

echo 'docker compose up -d && docker compose logs -f' > restart.sh && chmod +x restart.sh
docker compose stop && docker compose rm -f && ./restart.sh
docker exec -it cs1 curl https://bing.com -vv # 验证无法联网

docker inspect cs1 | grep IP # 看看ip

echo '172.31.0.2 xxx.example.com' | sudo tee -a /etc/hosts
curl http://xxx.example.com:8443/  # 加入hosts之后，本地验证
```

3. 加入hosts其实就是小trick的前奏，剩下的就是使用代理访问服务了。代理协议比https甚至更加注意隐私。相关的rule如下：

```
DOMAIN-SUFFIX,example.com,MYIP_PROXY,extended-matching
```
