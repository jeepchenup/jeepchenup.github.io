---
layout: post
title: "在 Ubuntu 中安装 RabbitMQ"
author: ABei
categories : Linux MQ
tags: Ubuntu RabbitMQ
---
* content
{:toc}

> Ubuntu 版本：16.04 LTS；RabbitMQ 版本：3.7.14




本文其实就是跟着官网走了一遍。对于 Ubuntu 和 Debian 这两个系统而言，RabbitMQ 官网推荐了 2 种方式来下载 RabbitMQ：

1.  通过 Package Cloud 或 Bintray 的 `apt` 仓库下载（推荐方式）
1.  下载压缩包然后手动安装。

这里我是使用的是 **Bintray** 上的 `apt` 仓库，好了废话不多说进入正题。

## 添加签名密钥

为了使用存储库，将 RabbitMQ 的签名密钥添加到 `apt-key` 中：

```shell
sudo apt-key adv --keyserver "hkps.pool.sks-keyservers.net" --recv-keys "0x6B73A36E6026DFCA"
##  不需要密钥服务器，密钥也可以被下载和引入进来
wget -O - "https://github.com/rabbitmq/signing-keys/releases/download/2.0/rabbitmq-release-signing-key.asc" | sudo apt-key add -
```

## 开启 apt HTTPS 传输

这个是为了确保 RabbitMQ 和 Erlang 包能从 Bintray 上下载，必须得确保你当前系统已经安装了 `apt-transport-https`。

```shell
sudo apt-get install apt-transport-https
```

## 添加资源列表文件

在 `/etc/apt/sources.list.d` 路径下，创建一个 `bintray.rabbitmq.list` 的文件。

其文件内容如下：

```shell
# Source repository definition example
# See below for supported distribution and component values
# This repository provides Erlang packages
deb https://dl.bintray.com/rabbitmq-erlang/debian xenial erlang
# This repository provides RabbitMQ packages
deb https://dl.bintray.com/rabbitmq/debian xenial main
```

## 下载

在完成上面的步骤之后，就可以开始下载 RabbitMQ 的工作了。

```shell
# 先进行更新（必要操作）
sudo apt-get update -y
# 开始下载 RabbitMQ 包
sudo apt-get install -y rabbitmq-server
```

上面的时间可能需要花点时间，请安心等待。安装完之后，RabbitMQ 是默认启动的。

## 查看 RabbitMQ 状态

可以通过 `ps -ef | grep rabbitmq` 或 `sudo service rabbitmq-server status`

等等这就完了吗？不能！一般，我们看到别人的 RabbitMQ 都有一个管理界面。下面我们来看看这个是怎么弄得。

## 开启管理界面

默认情况下，RabbitMQ 的管理界面是关闭的。我们需要手动的开启。

```shell
sudo rabbitmq-plugins enable rabbitmq_management
```

访问管理界面地址：`http://localhost:15672`。

> 账号: guest；密码：guest

可以看到下面的界面说明你已经成功的安装了 RabbitMQ。恭喜啊！！！！大兄弟。

![](http://cdn.51leif.com/2019-05-06_215140.png)

## 远程 IP 访问

你以为这样就结束了？当然不会啦！如果，我想要在其他机子上访问当前 Ubuntu 上面的 RabbitMQ 管理界面，那要怎么办呢？

聪明的你，可能已经想到了。

```shell
# 获取本机 ip 地址
ifconfig -a
```

![](http://cdn.51leif.com/2019-05-06_222224.png)

怎么会访问不了呢？经过一番查阅资料之后，发现在 RabbitMQ 从 `3.3.0` 开始禁止使用 `guest/guest` 权限通过除 **localhost** 外的 IP 访问。

如果想要开启远程 IP 访问，就需要手动修改配置文件 `rabbit.app` 文件。

```shell
# 查找 rabbit.app 文件
locate rabbit.app
```

将文件中的 `{loopback_users, [<<”guest”>>]}，` 改为 `{loopback_users, []}，`

```shell
# 最后重启 RabbitMQ SERVER 就能开启远程 IP 访问了
systemctl restart rabbitmq-server.service
```