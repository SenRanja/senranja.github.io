---
title: docker_cron
date: 2024-08-29 08:04:35
tags:
top: 27
categories:
- Docker
---

In the end of 2023, I developed a MCV web system and I found there was a problem: I want to use docker to build a container with crontab. But the crontab didn't work after build.

This is my before command, and it doesn't work:

    # 开启cron定时任务
    /etc/init.d/cron restart && /etc/init.d/cron status && /etc/init.d/cron start

This is my revision, and it works:

    # crontab
    RUN echo "* * * * * root cd /app/scripts/;/usr/local/bin/python /app/scripts/timer_add.py >> /var/log/timer.log 2>&1" >> /etc/cron.d/crontab && chmod 0644 /etc/cron.d/crontab && /etc/init.d/cron restart && /etc/init.d/cron status && chmod +x /app/start.sh

The critical element is the priviledges.

And I read the other solutions below, but I dont verify them.

`/var/spool/cron/[crontabs]/<username>`

**centos** `/var/spool/cron/root`

**Debian/Ubuntu** `/var/spool/cron/crontabs/root`
