---
title: RabbitMQ的基本使用
date: 2020-08-18 15:29:20
tags: 消息队列
---

之前用过activeMQ，zeroMQ，但是都是公司要用，并非自己选型，且从来没有考虑过消息幂等性问题，决定使用MQ就要承担使用MQ后的带来的系统复杂程度提高。看过好几个MQ对比，决定看看RabbitMQ和RocketMQ，今天先快速了解一下RabbitMQ。

# RabbitMQ快速入门

## 安装RabbitMQ

拉取RabbitMQ带web界面的镜像。`docker pull rabbitmq:management`

启动容器

```shell
docker run -d --restart=always --name rabbitmq -p 5672:5672 -p 6006:15672 -v /usr/project/docker/rabbitmq/data:/var/lib/rabbitmq rabbitmq:management
```

