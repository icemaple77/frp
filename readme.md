![使用 AWS + frp  让你的树莓派4实现免费内网穿透](./使用 AWS + frp 让你的树莓派 4 实现免费内网穿透 - 知乎\_files/v2-cbd509e7614e5d93cb7a7c1e8cbf6446_r.jpg)

# 使用 AWS + frp 让你的树莓派 4 实现免费内网穿透

试了很多内网穿透的方法。

目前主流的内网穿透服务/工具有：

- Ngrok
- Frp
- Natapp
- Lanproxy
- Spike
- 花生壳

像花生壳，Ngrok 这样的服务虽然简单易用，而且不需要自己准备公网 IP

但是有很多限制，速度也不是那么理想。

突然想到，利用 AWS EC2 12 个月的免费使用额度 + frp 来实现免(bai)费(piao)内网穿透，岂不美哉。

首先，在 AWS 控制台上开启一个新的 EC2 实例，注意要选择 centos7 、ubuntu 之类的免费 OS。

![](https://pic2.zhimg.com/v2-b5804d696c758e2d907ff38aeaeb1399_b.jpg)

![](./使用 AWS + frp 让你的树莓派 4 实现免费内网穿透 - 知乎\_files/v2-b5804d696c758e2d907ff38aeaeb1399_720w.webp)

注意实例类型要选择 t2.micro 其他的会收费哦

启动后用保存好的秘钥 SSH 连接实例

共有 DNS 可以在 AWS EC2 控制台上找到

    ssh -i "秘钥文件.pem" centos@公有 DNS (IPv4)

    例：

    ssh -i "key.pem" centos@ec2-54-185-35-352.us-west-2.compute.amazonaws.com

连接成功后，接下来分别在树莓派端和 AWS 服务器端下载 frp

[fatedier/frp​github.com/fatedier/frp/releases![](./使用 AWS + frp 让你的树莓派 4 实现免费内网穿透 - 知乎\_files/v2-a0e095d6b58fb22b9e1938f0ecc9e393_ipico.jpg)](https://link.zhihu.com/?target=https%3A//github.com/fatedier/frp/releases)

**树莓派端**：

1.下载 frp

    wget https://github.com/fatedier/frp/releases/download/v0.31.1/frp_0.31.1_linux_arm.tar.gz

2.解压并进入 frp 目录

    tar -xzvf frp_0.31.1_linux_arm.tar.gz
    cd frp_0.31.1_linux_arm

3.修改 frpc.ini 文件，并填写 EC2 实例分配的公网 IP 地址，假设我的 IP 为 54.185.35.252

![](https://pic4.zhimg.com/v2-682900bc4ea58ed7812fbfb5deea4e53_b.jpg)

![](./使用 AWS + frp 让你的树莓派 4 实现免费内网穿透 - 知乎\_files/v2-682900bc4ea58ed7812fbfb5deea4e53_720w.webp)

在控制台右下角找到自己的公网 IP

    # frpc.ini
    [common]
    server_addr = 54.185.35.252 #实例的公网IP地址
    server_port = 7000

    [ssh]
    type = tcp
    local_ip = 127.0.0.1
    local_port = 22
    remote_port = 6000

4.启动 frpc

    ./frpc -c ./frpc.ini

    # 想要后台启动的话可以用nohup命令，可以在frp.log上查看输出日志
    nohup ./frpc -c ./frpc.ini &

![](https://pic4.zhimg.com/v2-2c4bd35815987a4dae30a3df4920a27b_b.png)

![](./使用 AWS + frp 让你的树莓派 4 实现免费内网穿透 - 知乎\_files/v2-2c4bd35815987a4dae30a3df4920a27b_720w.webp)

成功开启 frps,如果出现 frps 启动失败的情况就换成 root 权限再试一次

树莓派端的准备工作完毕，接下来设置 AWS 服务器端的 frp

**AWS 服务器端：**

1.下载 frp

    wget https://github.com/fatedier/frp/releases/download/v0.31.1/frp_0.31.1_linux_amd64.tar.gz

2.解压并进入 frp 目录

    tar -xzvf frp_0.31.1_linux_amd64.tar.gz
    cd frp_0.31.1_linux_amd64

3.修改 frps.ini 文件

    # frps.ini
    [common]
    bind_port = 7000

4.启动 frps，这里可以选择用 nohup 命令让 frps 后台执行

    nohup ./frpc -c ./frpc.ini  &

成功启动服务后再开一个终端试试能否远程连接树莓派

    ssh -oPort=6000 pi@54.185.35.252

![](https://pic2.zhimg.com/v2-adf4569bd00dea779eab75fcb60a3391_b.jpg)

![](data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='1020' height='817'></svg>)

连接成功

以上就是 AWS+frp 免费实现内网穿透的教程了，

EC2 每个月有 750 个小时的免费额度，就算一天 24 小时不间断开启也不会被氪金，

需要注意的是 EC2 每次重启都会重新分配一个公网 IP，记得在树莓派端修改新的公网 IP。

}
