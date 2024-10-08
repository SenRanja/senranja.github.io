---
title: 构建blingTrendsAPIServer
date: 2024-04-09 17:58:30
tags:
categories:
- Docker
---

amd64/alpine:3.14足够小

    FROM amd64/alpine:3.14

    ADD . /api-server/

    RUN apk update && apk add tzdata && cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo "Asia/Shanghai" > /etc/timezone && apk del tzdata && apk add --update --no-cache python3 py3-pip && python3 -m pip install -r /api-server/requirements.txt

    WORKDIR /api-server

    EXPOSE 5000

    ENTRYPOINT ["python3", "/api-server/blingTrendsAPIServer.py"]



推荐dockerfile部署，目前已部署在192.168.3.203:5000。

默认使用5000端口，自行考虑防火墙端口开放，如ubuntu使用`sudo ufw allow 5000/tcp`放开端口；

目前连接数据库是`192.168.3.203:23306`，docker默认bridge模式，要访问203数据库需要`sudo ufw allow 23306/tcp`

如果需要指定端口映射，自行更改参数`docker run -p <host-port>:<container-port> <image-name>`

```
# 构建后大小为77.5MB
docker build -t bling-trends-server .

# 运行命令
docker run --name bling_trends_server_container -itd -p 5000:5000 bling-trends-server
```


