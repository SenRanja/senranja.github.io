---
title: 构建gitleaks-Docker服务
date: 2024-04-09 17:58:30
tags:
top: 1
categories:
- Docker
---

在上面gitleaks的docker-compose+Dockerfile的组合之后，我想写个纯docker-compose无dockerfile的版本。

即使用docker-compose完全替代Dockerfile。


docker-compose

entrypoint不好用，我查阅资料说是类似于Dockerfile的RUN，在容器启动之前的构件中会运行，但是怎么测试都不对。

干脆把RUN中的命令写到最后的command构建命令中。
涉及command命令，注意使用docker-compose logs看日志，有可能无限重复运行command命令

    version: "3"
    services:
    secretdetection_docker:
        image: amd64/alpine:3.14
        # entrypoint: ["/secretDetection/entrypoint.sh"]
        # entrypoint: "/bin/sh -c "apk update && apk add git && apk add tzdata && cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo "Asia/Shanghai" > /etc/timezone && apk del tzdata"
        container_name: secret_detection_http
        ports:
        - "8000:8000"
        restart: always
        volumes:
        - ./SecretDetectionDir:/secretDetection/
        restart: always
        working_dir: /secretDetection/
        # command: /secretDetection/http
        command: /bin/sh -c "apk update && apk add git && apk add tzdata && cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo "Asia/Shanghai" > /etc/timezone && apk del tzdata && /secretDetection/http"
        

