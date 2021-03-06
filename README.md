# 【WGCF】连接CF WARP为服务器添加IPv4/IPv6网络

* * *

# 目录

- [好处](README.md#好处)
- [为IPv6服务器添加IPv4网络接口方法](README.md#为IPv6服务器添加IPv4网络接口方法)
- [为IPv4服务器添加IPv6网络接口方法](README.md#为IPv4服务器添加IPv6网络接口方法)
- [临时、永久关闭和开启WGCF网络接口](README.md#临时永久关闭和开启WGCF网络接口)
- [EUserv 主机名变为 DiG9 不能正常使用 NAT64 解决办法](https://github.com/fscarmen/warp/tree/main/DiG9#euserv-%E4%B8%BB%E6%9C%BA%E5%90%8D%E5%8F%98%E4%B8%BA-dig9-%E4%B8%8D%E8%83%BD%E6%AD%A3%E5%B8%B8%E4%BD%BF%E7%94%A8-nat64-%E8%A7%A3%E5%86%B3%E5%8A%9E%E6%B3%95)
- [WARP原理](README.md#WARP原理)
- [致谢](README.md#致谢下列作者和项目排名不分先后)

* * *

## 好处

* 能让像EUserv这样的IPv6 only VPS上做的节点支持Telegram
* IPv6 建的节点能在 PassWall、ShadowSocksR Plus+ 上应用
* 解锁奈飞流媒体
* 可调用IPv4接口使京* docker 和 V2P 等正常运行
* 由于可以双向转输数据，能做对方VPS的跳板和探针，替代 HE tunnelbroker


## 为IPv6服务器添加IPv4网络接口方法

* 脚本会先行判断 EUserv 3种系统：Ubuntu 20.04、Debian 10、CentOS 8，再自动选相应的程序来完成，不需要人工选择。 

* 完成后看到有 wgcf 的网络接口即为成功，并自动清理安装时的临时文件。

```bash
echo -e "nameserver 2a00:1098:2b::1\nnameserver 2a03:7900:2:0:31:3:104:161" > /etc/resolv.conf
```
```bash
wget -N --no-check-certificate https://cdn.jsdelivr.net/gh/kkkyg/DJwarp/DJwarp.sh && chmod +x DJwarp.sh && ./DJwarp.sh
```



## 为IPv4服务器添加IPv6网络接口方法

* 脚本会先行判断 Oracle 3个系统：Ubuntu 20.04、Ubuntu 20.04 Minimal、CentOS 8，再自动选相应的程序来完成，不需要人工选择。 

* 完成后看到有 wgcf 的网络接口即为成功，并自动清理安装时的临时文件。

* 甲骨文 CentOS 8 由于更新内核需要时间约15分钟，停留在 Running scriptlet: kernel-core-4.18.0-240.15.1.el8_3.x86_64 的时候耐心等待即可。 

```bash
wget -N --no-check-certificate "https://raw.githubusercontent.com/fscarmen/warp/main/warp6.sh" && chmod +x warp6.sh && ./warp6.sh
```


## 临时、永久关闭和开启WGCF网络接口

临时关闭 wgcf（reboot重启后恢复开启） ```wg-quick down wgcf``` ，恢复启动 ```wg-quick up wgcf```

禁止开机启动 ```systemctl disable wg-quick@wgcf```,恢复开机启动 ```systemctl enable wg-quick@wgcf```


## WARP原理

WARP是CloudFlare提供的一项基于WireGuard的网络流量安全及加速服务，能够让你通过连接到CloudFlare的边缘节点实现隐私保护及链路优化。

其连接入口为双栈（IPv4/IPv6均可），且连接后能够获取到由CF提供基于NAT的IPv4和IPv6地址，因此我们的单栈服务器可以尝试连接到WARP来获取额外的网络连通性支持。这样我们就可以让仅具有IPv6的服务器访问IPv4，也能让仅具有IPv4的服务器获得IPv6的访问能力。

* 为仅IPv6服务器添加IPv4

原理如图，IPv4的流量均被WARP网卡接管，实现了让IPv4的流量通过WARP访问外部网络。

![2021-02-04_21-45-45.png](https://i.loli.net/2021/03/20/XesDmluhRBkHSjd.png)

* 为仅IPv4服务器添加IPv6

原理如图，IPv6的流量均被WARP网卡接管，实现了让IPv6的流量通过WARP访问外部网络。

![2021-02-04_21-45-44.png](https://i.loli.net/2021/03/20/6Tmc7WQOGa8dDoM.png)

* 双栈服务器置换网络

有时我们的服务器本身就是双栈的，但是由于种种原因我们可能并不想使用其中的某一种网络，这时也可以通过WARP接管其中的一部分网络连接隐藏自己的IP地址。至于这样做的目的，最大的意义是减少一些滥用严重机房出现验证码的概率；同时部分内容提供商将WARP的落地IP视为真实用户的原生IP对待，能够解除一些基于IP识别的封锁。

![2021-02-04_21-45-45-1.png](https://i.loli.net/2021/03/20/7vWf15szTONgq69.png)

* 网络性能方面：内核集成＞内核模块＞wireguard-go

Linux 5.6 以上内核则已经集成了 WireGuard ，可以用 ```hostnamectl```或```uname -r```查看版本。

甲骨文是 KVM 完整虚拟化的 VPS 主机，而官方系统由于版本较低，在不更换内核的前提下选择  "内核模块" 方案。

EUserv是 LXC 非完整虚拟化 VPS 主机，共享宿主机内核，不能更换内核，只能选择 "wireguard-go" 方案。
    

## 致谢下列作者和项目（排名不分先后）：  

技术指导:
* P3terx：https://p3terx.com/archives/use-cloudflare-warp-to-add-extra-ipv4-or-ipv6-network-support-to-vps-servers-for-free.html
* 甬哥探世界：https://www.youtube.com/watch?v=78dZgYFS-Qo
* Luminous：https://luotianyi.vc/5252.html

所需文件：
* wgcf：https://github.com/ViRb3/wgcf
* WireGuard-GO 编译自官方：https://git.zx2c4.com/wireguard-go/
* 中科大 elrepo 源：https://www.dazhuanlan.com/2020/04/02/5e8524f06823d/
