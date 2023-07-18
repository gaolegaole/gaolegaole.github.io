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
`docker run -d --name openfrp -p 15422:22 ubuntu sh -c "tail -f /dev/null"`
最后的tail -f是为了让docker运行后不退出
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

# 通过ssh连接
`ssh root@frphost -p 10022 `
frphost是启动frp后会出现的地址，-p是外网端口

![1689647444310.png](./post/ubuntu_sshd/1689647444310.png)

# 配置systemctl
1. 安装systemd `apt install systemd -y`
2. 启动 systemd //TODO 不知道怎么启动😂
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


