<!--
 * @Author: your name
 * @Date: 2020-04-24 10:22:50
 * @LastEditTime: 2020-04-24 22:17:06
 * @LastEditors: Please set LastEditors
 * @Description: In User Settings Edit
 * @FilePath: \git_study\docker-study\docker和WSL2的挖坑录.md
 -->

# docker 和 WSL2 以及配置代理(给自己挖坑)

## 前提条件

- WSL2, 请确认WSL运行的版本为2, `wsl -l -v`
- docker for desktop(chancel: edge), 请确认开启了WSL2配置选项
- v2rayn 等一众代理工具

## 开始挖坑

WSL2 和 docker 的组合很妥帖的感觉, 但是这个代理真的头疼, [微软官方文档对 WSL2 的网络介绍](https://docs.microsoft.com/zh-cn/windows/wsl/wsl2-ux-changes#accessing-network-applications)中有以下相关内容:

文档中对网络介绍分为`从WSL2联系Windows`, `Windows联系WSL2`, `外界远程访问WSL2` 和 `从LAN访问WSL2`四个方面:

1. WSL2联系Windows
   使用`cat /etc/resolv.conf` 显示 `namesever` 信息
   确认好端口(即本地代理工具的端口)即可访问
  
   - WSL2 终端中: `export https_proxy="http://<namesever>:port"` `export https_proxy="http://<namesever>:port"`
   - Docker for Desktop 的配置中: 设置PROXIES中 Http_proxy 和 Https_proxy 均为`namesever>:port`
   - container 中: 类似于WSL2终端配置, 然后注意配置 git 的代理

2. Windows联系WSL
   在18945版本之后, 都可以直接使用localhost访问
   18945 版本之前的, 请自行查看上述链接

3. 外界远程访问WSL2
   可以将远程的访问认为是LAN的访问进行处理, 不过要注意WSL2中的端口和允许IP需要开放

4. 从LAN访问WSL2
   类似于VM虚拟机的操作方式, 也需要使用WSL2的 `ip addr | grep eth0` 中查到的 `inet` 来访问

### 注意

需要设置全 `All_proxy` `Http_proxy` `Https_proxy` 设置教程搜索引擎自寻, 比较简单
**!!请务必记得对防火墙进行开放, WSL2的网络访问都是相当于局域网的, 如果防火墙拦截, 会出现timeout的问题(博主血泪史)**

### 测试

WSL2 的网络着实有些问题, 如果上述都配置好了, 可以通过 node.js, 在Windows终端配置好一个http sever, 代码如下

```JavaScript
var http = require('http');

http.createServer(function (request, response) {
    response.writeHead(200, {'Content-Type': 'text/plain'});

    if (request) {
        console.log('receive a request')
    }
    response.end('Hello World\n');
}).listen(8000);

// 终端打印如下信息
console.log('Server running at http://127.0.0.1:8000/');
```

然后在WSL2, docker container 中 通过
`curl -vv <namesever>:8000` 查看能否返回 `Hello World`
`curl -vv https://google.com` 查看代理是否连通

## 实践用例

本身是一个课程需要使用 lean 的 OpenWrt 仓库来生成固件的, 最后琢磨两种解决方案

### 使用 WSL2 支持的Docker

可以使用两种方法生成镜像

#### 自己写(噗嗤)

过程(以下内容均可以在 `PowerShell Core 7.0.2 preview` 中实现)
注意: 所有的内容都已经过代理配置(且终端中实现的代理都需要在终端重新进入时候, 重新配置)

1. `docker pull ubuntu:18.04`
2. `docker run -it ubuntu:18.04`
3. 在container中执行以下代码

   ```bash
   apt update
   // 使apt支持https方便之后使用清华源镜像加速(U1S1, docker里的官方Ubuntu镜像太精简了)
   apt install apt-transport-https ca-certificates
   mv /etc/apt/sources.list /etc/apt/sources_old.list
   ```

4. 从[清华源镜像](https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/)上准备Ubuntu 中 apt 的 `sources.list` 文件, 博主弄在了 `F:\\sources.list` 中

5. 通过 docker 将 sources 文件传到 container 中
   - 终端执行: `docker cp F:\\sources.list <CONTAINER ID>:/etc/apt/sources.list`
   - `F:\\sources.list` 替换成自己准备好的清华源镜像文件
   - CONTAINER ID: 通过 `docker ps -a` 寻找对应的ID

6. 在container中执行以下代码

   ```bash
   apt update
   apt upgrade
   apt install sudo git vim
   adduser <username>
   // 输入好密码后
   visudo
   // 在 root    ALL=(ALL:ALL) ALL 下面添加
   <username>    ALL=(ALL:ALL) ALL
   // Ctrl+O 回车 保存
   su <username>
   cd ~
   sudo apt-get -y install build-essential asciidoc binutils bzip2 gawk gettext git libncurses5-devlibz-dev patch python3.5 unzip zlib1g-dev lib32gcc1 libc6-dev-i386 subversion flex uglifyjs git-coregcc-multilib p7zip p7zip-full msmtp libssl-dev texinfo libglib2.0-dev xmlto qemu-utils upx libelf-devautoconf automake libtool autopoint device-tree-compiler g++-multilib antlr3 gperf
   // 键入密码
   git clone https://github.com/coolsnowwolf/lede
   cd lede
   ./scripts/feeds update -a
   ./scripts/feeds install -a
   make menuconfig
   // 自己生成配置文件
   make -j8 download V=s
   make -j1 V=s
   ```

7. 编译完之后在Powershell中执行 `docker cp <CONTAINER ID>/home/<username>/ F:\\`

完成!

#### 使用Docker Hub 上已经写好的镜像

##### 使用 timiil 大神的

[镜像地址](https://hub.docker.com/r/timiil/coolsnowwolf-lede-builder/)

1. PowerShell: `docker run -it -v /home/lede_output:/lede/bin timiil/coolsnowwolf-lede-builder`
2. Container: 

   ```bash
   make menuconfig // 自己生成配置文件
   make -j8 download V=s
   make -j1 V=s
   ```

3. 编译完之后在Powershell中执行 `docker cp <CONTAINER ID>/home/<username>/ F:\\`

##### 使用 博主 caiji的

[镜像地址]()

<!-- TODO dockerfile 稍后补充 -->
1. PowerShell
2. Container

   ```bash
   make menuconfig // 自己生成配置文件
   make -j8 download V=s
   make -j1 V=s
   ```

3. 编译完之后在Powershell中执行 `docker cp <CONTAINER ID>/home/<username>/ F:\\`

### 使用 hyper-V 支持的Docker

这个更推荐: 毕竟WSL2和Docker结合还是实验性质的问题很多, 比如跑着跑着就Timeout....

不具体解释了, 需要注意一下几点

- 配置好镜像的存储位置, 在 docker -> setting -> Resources -> ADVANTED -> Disk image location
- 配置好代理, 因为是相当于生成了一个虚拟机, 所以就配置成host主机的ip地址(通过config查看), 端口是代理软件的端口

总之呢, WSL2 和 Docker 的融合比较艰难, 还是等着巨硬多完善一下吧, 还有就是遇到问题多看文档, 很多时候博客什么的内容还是不够全面

如果大家有什么问题可以发邮件联系我: 2646677495@qq.com