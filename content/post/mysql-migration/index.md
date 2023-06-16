---
title: "记录一次mysql迁移"
date: 2023-06-16T15:55:56+08:00
draft: false
description: "记录一次mysql迁移"
categories:

tags:
    - mysql
    - docker
    - graphjin
---
> 背景：项目中需要使用graphjin这个包，由于数据库是mysql5.7，而这个库最低都需要mysql8或者postgresql，所以要么升级数据库，要么切换数据库中间件。\
> graphjin：是一个可以自动生成graphql的包，方便低代码类平台开发，可以自定义数据对象等，而不需要写crud

# 1. docker pull mysql：8.0.33
这个镜像不知道怎么回事，换了好几个源都无法下载，使用了8.0则可以了。
## 切换源系统不一样，方式也有区别，看第二
切换源：[切换源方法](https://yeasy.gitbook.io/docker_practice/install/mirror)\
阿里云：[阿里云修改容器方法](https://help.aliyun.com/document_detail/60750.html?spm=5176.smartservice_service_robot_chat_new.0.0.1764709acxKRSU)
## 修改/etc/docker/daemon.json后无法启动docker
需要将那个文件移除，可能是由于我的系统是Centos7，那个文件不需要或者格式有问题等，移除后启动正常。

# 2. 备份docker中的mysql数据
个人项目，没有挂载数据目录。进入docker容器后执行：`mysqldump -u username -p --databases db1 > /tmp/db1.sql`输入密码后会导出到/tmp目录下
# 3. 导入db1.sql
## 1. 停掉旧的容器
`docker stop db1`
## 2. 创建容器
`docker run -d --name db1 -p 3306:3306 -e MYSQL_DATABASE=db1 -e MYSQL_USER=db1 -e MYSQL_PASSWORD=123456 -e MYSQL_ROOT_PASSWORD=123456 mysql:8.0 --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci`
## 3. 同样进入容器后执行
`mysql -u username -p db1 < /tmp/db1.sql`
