---
title: 安装docker方法
tags: [docker]
date: 2020-03-10 10:00:00
---

### 一、课程方式安装Docker



```shell
# 1、yum 包更新到最新 
yum update
# 2、安装需要的软件包， yum-util 提供yum-config-manager功能，另外两个是devicemapper驱动依赖的 
yum install -y yum-utils device-mapper-persistent-data lvm2
# 3、 设置yum源
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
# 4、 安装docker，出现输入的界面都按 y 
yum install -y docker-ce
# 5、 查看docker版本，验证是否验证成功
docker -v
```

### 二、其他安装方式(推荐)

教程链接：https://www.jianshu.com/p/1e5c86accacb