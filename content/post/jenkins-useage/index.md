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
    - net
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
[jenkins doc https://www.jenkins.io/doc/book/security/services/](https://www.jenkins.io/doc/book/security/services/)
