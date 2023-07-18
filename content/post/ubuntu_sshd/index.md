---
title: "Ubuntu_sshd"
date: 2023-07-18T10:16:32+08:00
draft: false
description: "Ubuntu_sshd"
categories:

tags:
    - sshd
    - linux
---
# 启动docker
1. `docker run -d --name openfrp -p 15422:22 ubuntu sh -c "tail -f /dev/null"`

最后的tail -f是为了让docker运行后不退出

2. sys info
```shell
root@d4b356ecbaed:/# cat /etc/lsb-release
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=22.04
DISTRIB_CODENAME=jammy
DISTRIB_DESCRIPTION="Ubuntu 22.04.2 LTS"
```
# 安装必要软件
```shell
apt update && apt install wget iproute2 iputils-ping vim openssh-server -y
```
# 安装frp客户端
1. wget https://sq.oss.imzzh.cn/client/OpenFRP_0.49.0_5cc2e1cc_20230618/frpc_linux_amd64.tar.gz
2. gzip -d frp_linux_amd64.tar.gz
3. tar -xvf frp_linux_amd64.tar
4. chmod +x frp_linux_amd64
5. mv frp_linux_amd64 /usr/sbin/openfrp
# 启动ssh
## 1. 修改/etc/ssh/sshd_config
```shell
...
Port 22
AddressFamily any
ListenAddress 0.0.0.0
...
PermitRootLogin yes
...
PasswordAuthentication yes
```
大概就这些需要修改地方 [Reference https://askubuntu.com/questions/1378153/ssh-permission-denied](https://askubuntu.com/questions/1378153/ssh-permission-denied)

## 2. 启动ssh
`/etc/init.d/ssh start //未配置systemd这样启动`
# 启动frp
1. 在官网openfrp.net下载配置文件，保存到/frpc.ini中
2. /usr/sbin/openfrp -c /frpc.ini

![1689647360303.png](./post/ubuntu_sshd/1689647360303.png)
# 修改docker 默认密码
```shell
passwd
```
# 通过ssh连接
`ssh root@frphost -p 10022 `
frphost是启动frp后会出现的地址，-p是外网端口

![1689647444310.png](./post/ubuntu_sshd/1689647444310.png)

# 配置systemctl
1. 安装systemd `apt install systemd -y`
2. 启动 systemd //TODO 不知道怎么启动😂，container不建议使用systemd。[参考：https://stackoverflow.com/questions/59466250/docker-system-has-not-been-booted-with-systemd-as-init-system](https://stackoverflow.com/questions/59466250/docker-system-has-not-been-booted-with-systemd-as-init-system)[参考：https://stackoverflow.com/questions/59466250/docker-system-has-not-been-booted-with-systemd-as-init-system](https://stackoverflow.com/questions/59466250/docker-system-has-not-been-booted-with-systemd-as-init-system)
3. systemctl enable ssh && systemctl start ssh
4. frp service [参考：https://medium.com/@benmorel/creating-a-linux-service-with-systemd-611b5c8b91d6](https://medium.com/@benmorel/creating-a-linux-service-with-systemd-611b5c8b91d6)
    * touch /etc/systemd/system/openfrp.service
    * 内容
    ```shell
    [Unit]
    Description=ROT13 demo service
    After=network.target
    StartLimitIntervalSec=0
    [Service]
    Type=simple
    Restart=always
    RestartSec=1
    ExecStart=/usr/sbin/openfrp -c /frpc.ini

    [Install]
    WantedBy=multi-user.target
    ```
    [参考：https://medium.com/@benmorel/creating-a-linux-service-with-systemd-611b5c8b91d6](https://medium.com/@benmorel/creating-a-linux-service-with-systemd-611b5c8b91d6)
    * systemctl enable openfrp
    * systemctl start openfrp
# 为了安全使用key登录
## 1. 生成key
```shell
ssh-keygen -t rsa
```
默认是将key保存在~/.ssh目录下，id_rsa 和 id_rsa.pub
## 2. 将私钥导出
```shell
docker cp open:/root/.ssh/id_rsa d:/code/id_rsa
```
## 3. 把公钥加到~/.ssh/authorized_keys中
```shell
cat ~/.ssh/id_rsa >> ~/.ssh/authorized_keys
```
## 4. 登录测试
```shell
ssh root@frphost -p port -i d:/code/id_rsa
```
output
```shell
PS D:\code> ssh root@jp-osk-bgp-1.openfrp.top -p [端口] -i D:\code\id_rsa
Welcome to Ubuntu 22.04.2 LTS (GNU/Linux 5.10.16.3-microsoft-standard-WSL2 x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, you can run the 'unminimize' command.
Last login: Tue Jul 18 02:51:41 2023 from 127.0.0.1
root@d4b356ecbaed:~#
```
[Reference https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server](https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server)
# 错误
> Permissions for 'D:\\code\\id_rsa' are too open.

拷贝出来的私钥权限太大了，解决方法参考。[Reference https://superuser.com/questions/1296024/windows-ssh-permissions-for-private-key-are-too-open](https://superuser.com/questions/1296024/windows-ssh-permissions-for-private-key-are-too-open)

