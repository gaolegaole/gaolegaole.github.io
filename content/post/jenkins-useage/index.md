---
title: "流水线工具jenkins使用"
date: 2023-07-26T08:53:50+08:00
draft: false
description: "Jenkins Useage"
categories:

tags:
    - jenkins
    - DevOps
---
# docker-compose.yml
```yml
version: "3"
services:
    jenkins:
        container_name: jenkins
        image: jenkins/jenkins
        ports:
            - "8080:8080"
        volumes:
            - "$PWD/jenkins_data/jenkins_home:/var/jenkins_home"
        networks:
            - net
networks:
    net:
```
[network reference https://docs.docker.com/compose/networking/](https://docs.docker.com/compose/networking/)\
[docker image repo https://hub.docker.com/_/jenkins/](https://hub.docker.com/_/jenkins/)
# run container
```bash
docker-compose up -d
```
# troubleshooting
* docker permisson denied 

`sudo usermod -a -G docker jenkins` 。[Reference https://www.theserverside.com/blog/Coffee-Talk-Java-News-Stories-and-Opinions/Fix-Jenkins-Docker-Permission-denied-daemon-error](https://www.theserverside.com/blog/Coffee-Talk-Java-News-Stories-and-Opinions/Fix-Jenkins-Docker-Permission-denied-daemon-error)
* path permission
```bash
id
//output
PC0% id
uid=1000(zlbao) gid=1000(zlbao) groups=1000(zlbao),4(adm),20(dialout),24(cdrom),25(floppy),27(sudo),29(audio),30(dip),44(video),46(plugdev),117(netdev)
sudo chown /var/jenkins_home 1000 -R
```
Add Multiple Supplementary/Secondary Groups using Linux usermod command

You can add multiple Supplementary groups OR Secondary groups to a user account. You can do so by using usermod command with argument -G. Here we are using an extra argument i.e. -a which is also referred to as append.

Note: If you don't use the argument -a with -G then Linux usermod will remove all previously added Secondary groups and only add the current one which is you are adding.

```bash
[root@localhost ~]# usermod -a -G developers,workers ricky  # Add Supplementary/ Secondary to a User Account

# Confirm the Setting

[root@localhost ~]# id ricky
uid=555(ricky) gid=502(john) groups=502(john),557(developers),558(workers)
```
[reference https://www.itsmarttricks.com/best-linux-usermod-command-with-examples/](https://www.itsmarttricks.com/best-linux-usermod-command-with-examples/)\

# get default administor password
```bash
docker logs -f jenkins

//output
...
2023-07-26 01:29:14.605+0000 [id=40]    INFO    jenkins.InitReactorRunner$1#onAttained: Loaded all jobs
2023-07-26 01:29:14.607+0000 [id=40]    INFO    jenkins.InitReactorRunner$1#onAttained: Configuration for all jobs updated
2023-07-26 01:29:14.656+0000 [id=53]    INFO    hudson.util.Retrier#start: Attempt #1 to do the action check updates server
WARNING: An illegal reflective access operation has occurred
WARNING: Illegal reflective access by org.codehaus.groovy.vmplugin.v7.Java7$1 (file:/var/jenkins_home/war/WEB-INF/lib/groovy-all-2.4.21.jar) to constructor java.lang.invoke.MethodHandles$Lookup(java.lang.Class,int)
WARNING: Please consider reporting this to the maintainers of org.codehaus.groovy.vmplugin.v7.Java7$1
WARNING: Use --illegal-access=warn to enable warnings of further illegal reflective access operations
WARNING: All illegal access operations will be denied in a future release
2023-07-26 01:29:14.975+0000 [id=30]    INFO    jenkins.install.SetupWizard#init:

*************************************************************
*************************************************************
*************************************************************

Jenkins initial setup is required. An admin user has been created and a password generated.
Please use the following password to proceed to installation:

307f9e2870ab494f91928afcbf78b383

This may also be found at: /var/jenkins_home/secrets/initialAdminPassword
```
# how to limit memory and CPU useage of docker containers?
```bash
docker run -it --memory="1g" --memory-swap="2g" --cpus="1.0" ubuntu
```
> Note: If you don’t want to use swap memory, give --memory and --memory-swap the same values.
[reference https://phoenixnap.com/kb/docker-memory-and-cpu-limit](https://phoenixnap.com/kb/docker-memory-and-cpu-limit]\

# first job
1. 创建job后添加参数：NAME ,SHOW
2. 创建脚本
```bash
#!/bin/bash

NAME=$1   # 获取启动的第一个参数，$0是命令本身（script.sh)
SHOW=$2   # 第二个参数

if [ "$SHOW" = "true" ] ; then
    echo "Hello , $NAME"
else
    echo "Not show , SHOW is $SHOW"
fi
```
3. 添加shell
`/var/jenkins_home/script.sh $NAME $SHOW`
[reference https://geekdudes.wordpress.com/2019/08/02/jenkins-run-shell-script-add-parameters-to-job/](https://geekdudes.wordpress.com/2019/08/02/jenkins-run-shell-script-add-parameters-to-job/)\

# 创建一个远程主机
```dockerfile
FROM centos:7

RUN yum -y install openssh-server

# 创建用户后，设置密码，不使用交互模式，使用管道符号。将创建的目录配置只允许remote_user读写
RUN useradd remote_user && \
    echo "1234" | passwd remote_user --stdin && \
    mkdir -p /home/remote_user/.ssh && \
    chmod 700 /home/remote_user/.ssh 
# 使用ssh-keygen -f remote_user创建key文件，复制公钥到docker中
COPY remote-key.pub /home/remote_user/.ssh/authorized_keys
# 确保目录的所属
RUN chown remote_user:remote_user -R /home/remote_user/.ssh/ && \
    chmod 600 /home/remote_user/.ssh/authorized_keys

RUN /usr/sbin/sshd-keygen  #

CMD /usr/sbin/sshd -D #以后台方式启动sshd
```
修改docker-compose.yml
```yml
version: "3"
services:
    jenkins:
        ...
    remote-host:
        container_name: remote-host
        image: remote-host
        build:
            context: centos7 #文件夹下有Dockerfile文件
        networks:
            - net
```
完整的yml
```yml
version: "3"
services:
  jenkins:
    container_name: jenkins
    image: jenkins/jenkins
    volumes:
      - "d:\\code\\jenkins:/var/jenkins_home"
    ports:
      - "8080:8080"
      - "50000:50000"
    networks:
      - net
  remote_host:
    container_name: remote_host
    image: remote_host
    build:
      context: ./
    networks:
      - net
networks:
  net:
```
# build
1. `docker-compose build`
2. `docker-compose up -d`
# 使用remote_user登录remote_host
```bash
docker cp remote_user jenkins:/tmp/remote_user #拷贝私钥文件到jenkins
docker exec -it jenkins bash


cd /tmp
ssh remote_user@remote_host -i remote_user 

```
[jenkins doc https://www.jenkins.io/doc/book/security/services/](https://www.jenkins.io/doc/book/security/services/)
