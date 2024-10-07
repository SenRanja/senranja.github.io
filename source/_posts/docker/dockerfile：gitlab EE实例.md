---
title: gitlab EE实例
date: 2024-04-09 17:58:30
tags:
top: 18
categories:
- Docker
---


    version: '3.8'

    services:
    gitlab:
        image: 'gitlab/gitlab-ee:15.3.5-ee.0'
        container_name: 'gitlab-ee'
        restart: always
        hostname: 192.168.3.199
        privileged: true
        shm_size: '2gb'
        environment:
        GITLAB_OMNIBUS_CONFIG: |
            external_url 'http://192.168.3.199:40080'
            gitlab_rails['gitlab_shell_ssh_port'] = 40022
        ports:
        - 40080:40080
        - 40443:40443
        - "40022:22"
        volumes:
        - './storage/config:/etc/gitlab'
        - './storage/logs:/var/log/gitlab'
        - './storage/data:/var/opt/gitlab'
