# Linux如何使用Trojan协议

## Ubuntu Trojan客户端使用教程

此教程使用的是 Ubuntu 18.04 x64

### 1. 开启Trojan代理服务

* 使用此命令下载Trojan客户端

官方版本(GitHub)：cd /usr/src && wget https://github.com/trojan-gfw/trojan/releases/download/v1.15.1/trojan-1.15.1-linux-amd64.tar.xz

* 解压Trojan文件

tar xvf trojan-1.15.1-linux-amd64.tar.xz

* 打开配置文件

cd /usr/src/trojan
vi config.json
按i进入编辑模式
run_type 修改为 "client"
local_port 修改为 1080
remote_addr 修改为 jpo123.ovod.me
remote_port 修改为 22201
password 修改为 ["123456"]（节点的密码）
示例如下
```
    "run_type": "client",
    "local_addr": "0.0.0.0",
    "local_port": 1080,
    "remote_addr": "abc.server.me",
    "remote_port": 22201,
    "password": ["123456"]
```

ssl中的 verify 值修改为 false （如果配置文件中没有，则添加这个配置）
ssl中的 verify_hostname 值修改为 false （如果配置文件中没有，则添加这个配置）
ssl中的 cert 修改为 "" （改成空的）
示例如下
```
        "verify": false,
        "verify_hostname": false,
        "cert": "",
```

* 按ESC键退出编辑，输入:wq保存配置文件

* 使用以下命令配置 trojan service

```
cat > /etc/systemd/system/trojan.service <<-EOF
[Unit]
Description=trojan
After=network.target

[Service]
Type=simple
PIDFile=/usr/src/trojan/trojan.pid
ExecStart=/usr/src/trojan/trojan -c /usr/src/trojan/config.json -l /usr/src/trojan/trojan.log
ExecReload=/bin/kill -HUP \$MAINPID
Restart=on-failure
RestartSec=1s

[Install]
WantedBy=multi-user.target

EOF
```

* 启动Trojan

systemctl start trojan

* 检查是否启动成功

ps aux | grep trojan | grep -v grep

看到有类似 /usr/src/trojan/trojan 的内容展示，即表示trojan正在运行

如果未启动成功，通过这个命令查看日志： cat /usr/src/trojan/trojan.log

还可以执行 `curl ip.sb --socks5 127.0.0.1:1080`, 查看结果是否为Trojan代理的IP

* 如何设置为开机启动？

使用此命令：systemctl enable trojan

<hr>

### 2. 命令行使用Trojan代理

* linux中很多操作是在终端中进行

很多程序和服务的下载都需要通过 npm, gem, nvm, git等命令进行，而在国内下载速度较差，如果中断还要重新开始，通过全局翻墙可以改善这种情况。

* 安装配置proxychains

全局翻墙通过proxychains实现，即将任何程序和Trojan的proxy建立链接，原理和浏览器的代理相似

下载: `sudo apt-get install proxychains`

配置: `sudo vi /etc/proxychains.conf`

在最后的ProxyList里注释默认的socks代理： socks4 前增加#表示注释

在最后的ProxyList里加入Trojan的代理设置： `socks5 127.0.0.1 1080`

测试本地IP: `curl -4 ip.sb`，将显示自己的IP

测试代理IP: `proxychains curl -4 ip.sb`，将显示Trojan代理的IP

后续使用的命令行需要代理时，只需要在前面加上 `proxychains` 即可

如 `proxychains npm install`