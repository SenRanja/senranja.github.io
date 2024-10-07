---
title: 构建gitleaks-Docker服务
date: 2024-04-09 17:58:30
tags:
top: 19
categories:
- Docker
---

两个一块儿用，Dockerfile在docker-compose.yaml的同目录下。

# Dockerfile

    FROM amd64/alpine:3.14

    RUN apk update && apk add git && apk add tzdata && cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo "Asia/Shanghai" > /etc/timezone && apk del tzdata

    ADD ./SecretDetectionDir /webscan/

    WORKDIR /webscan

    EXPOSE 8000

    ENTRYPOINT ["/webscan/http"]

# docker-compose.yaml

    version: "3"
    services:
    secretdetection_docker:
        build: .
        container_name: secretdetection_http
        ports:
        - "8000:8000"
        restart: always
    

