+++
title = "打通和朋友宿舍间的网段"
date = "2023-12-02T17:23:53+08:00"
tags = ["Networking", "Random Stuff", "Router", "Linux"]
keywords = ["Wireguard", "OpenWRT", "Networking"]
description = "我想随时都能访问朋友宿舍的NAS，所以我打通了我们的网段。"
showFullContent = false
draft = false
+++

## 0x00 前言

### 干啥呢

我和朋友的宿舍里都有`路由器`，都开启了`NAT`，所以我们的网段是隔离的。

*不过`路由器`都是为了`多播`而存在的 （x*

朋友在他宿舍里放了个台式机，跑的是`TrueNAS` ~~(富哥)~~。他的`NAS`上面有一些虚拟机，我想直接访问它们。

### 一点别的事情

说来，上学期我测试的时候，宿舍楼的校园网是无法相互访问的，估计是开了`VLAN Isolation`啥的。不过这个学期突然又可以了，挺好的。

这个方案其实一开始我是使用`n2n`，通过自建的`n2n Supernode`进行打洞、通讯的。不过`n2n`似乎只能跑到`100Mbps`。我们的校园网接入是`100Mbps`，我猜测咱的`edge`节点走校园网的出口进行通讯了，而不是直接在校园网网段内通讯，所以速度很慢。

反正挺神奇的。

## 0x01 DDNS 以及一些别的

我需要知道对方宿舍路由器的`IP`地址。
这应该是很简单的事情，直接用`DDNS`就好了，我用的是`ddns-go`。

后来我尝试在本地解析对方的`IP`地址，却发现怎么查也查不出来。我还以为是解析没那么快，后来调查了一下才发现有个东西叫`重绑定保护`，在`OpenWrt`中是默认启用的，它会`丢弃包含 RFC1918 地址的上游响应`，也就是会丢弃掉DNS查询返回中的`私有地址`。

> RFC1918 你可以看 [这里](https://datatracker.ietf.org/doc/html/rfc1918)。在互联网的地址架构中，专用网络是指遵守RFC 1918和RFC 4193规范，使用专用IP地址空间的网络。私有IP无法直接连接互联网，需要使用网络地址转换或者代理服务器来实现。与公网IP相比，私有IP是免费的，同时节省了IP地址资源，适合在局域网使用。(来自[Wikipedia](https://zh.wikipedia.org/zh-cn/%E4%B8%93%E7%94%A8%E7%BD%91%E7%BB%9C))

这个东西是为了防止`DNS重绑定攻击`的。基本上就是，保护你的内网设备不被恶意网页访问到。如果你
感兴趣，你可以读读下面的内容：

> DNS重新绑定是计算机攻击的一种形式。 在这种攻击中，恶意网页会导致访问者运行客户端脚本，攻击网络上其他地方的计算机。 从理论上讲，同源策略可防止发生这种情况：客户端脚本只能访问为脚本提供服务的同一主机上的内容。 比较域名是实施此策略的重要部分，因此DNS重新绑定通过滥用域名系统（DNS）来绕过这种保护。
这种攻击可以通过让受害者的网络浏览器访问专用IP地址的机器并将结果返回给攻击者来破坏专用网络。 它也可以用于使用受害者机器发送垃圾邮件，分布式拒绝服务攻击或其他恶意活动。
(来自[Wikipedia](https://zh.wikipedia.org/wiki/DNS%E9%87%8D%E6%96%B0%E7%BB%91%E5%AE%9A%E6%94%BB%E5%87%BB))

解决方法就是，去 `OpenWrt` -> `网络` -> `DHCP/DNS` -> `常规设置` ->  `域名白名单`， 添加你的域名，之后你就可以解析到你的`DDNS`地址了。

## 0x02 WireGuard 与 NAT 与 IP Forward

理论上你可以直接写路由表来解决这个事情，但是我不想在内网里面裸奔，所以我还是用了`WireGuard`。

__首先两边内网里的两台及其能够通讯。__

这里把我这的机器叫做`Cat`，对方的机器叫做`Neko`。
（没有用路由器跑是因为，路由器的MT7621不太能跑满1000Mbps，效果在200Mbps的样子。）

|Device|IP Address|WireGuard Peer Address|
|:---:|:---:|:---:|
|Cat|192.168.1.233|10.66.124.2|
|Neko|192.168.124.233|10.66.124.1|

我先在两边的内网机器上配置了`WireGuard`。两边大概是这样：

- Neko

```ini
[Interface]
Address = 10.66.124.2/32
PrivateKey = {PrivateKeyForNeko}

[Peer]
AllowedIPs = 192.168.124.0/24, 10.66.124.0/24
Endpoint = {YourDDNSDomain}:{WireGuardPort}
PersistentKeepalive = 25
PreSharedKey = {PreSharedKey}
PublicKey = {PublicKeyForCat}
```

- Cat

```ini
[Interface]
Address = 10.66.124.1/32
ListenPort = {WireGuardPort}
PrivateKey = {PrivateKeyForCat}

[Peer]
AllowedIPs = 192.168.1.0/24, 10.66.124.0/24
PublicKey = {PublicKeyForNeko}
PresharedKey = {PresharedKey}
```

这里还需要在路由器上把`Neko`的`WireGuardPort`转发出来，以便让`Cat`连接到。

这样，`Neko`应该可以和`Cat`通讯了。

__之后是要让`Neko`和`Cat`能够通过对方访问对方的网段__

这就涉及到`IP转发`(`ip_forward`)了。

> 操作系统拥有IP转发功能，意味着该系统能接收从接口传送进来的网络数据包，如果识别到该数据包不用于该系统自身，那么系统将会将该网络数据包传送到另外一个网络去，用恰当地方式转发该数据包。 一个典型的场景是需要去搭建一个路由器以连接两个不同网络。
(来自[WikiPedia](https://zh.wikipedia.org/zh-cn/IP%E8%BD%AC%E5%8F%91))

由于需要从`WireGuard`的接口访问，比如说，`eth0`，就跨接口了。需要调整一个内核
参数来启用`IP转发`。

你需要在`/etc/sysctl.conf`中添加一行：

```conf
net.ipv4.ip_forward=1
```

然后执行`sysctl -p`来使其生效。

这样，内核就可以帮我们转发数据包了。

这里就需要配置`iptables`，来做`NAT`。

- Cat & Neko

```bash
iptables -t nat -A POSTROUTING -s 10.66.124.0/24 ! -d 10.66.124.0/24 -j MASQUERADE
```

> `MASQUERADE` 是一种 `SNAT`，它应该叫做`地址伪装`。它基本上做的事情就是智能的`SNAT`，它一般用于将局域网与互联网连接起来。具体你可以看[这里](http://www.vorkon.de/VorKon-12.1-Leseprobe/drittanbieter/Dokumentation/openSUSE_11.4/manual/cha.security.firewall.html)。如果你对`NAT`感兴趣，你可以去[这里](https://zh.wikipedia.org/wiki/%E7%BD%91%E7%BB%9C%E5%9C%B0%E5%9D%80%E8%BD%AC%E6%8D%A2)看看。

对了，你还需要将`iptables`规则持久化。基本上就是，你要执行类似这样的东西：

```bash
iptables-save > /etc/iptables/rules.v4
```

这样，`Neko`应该可以访问`Cat`所在内网的内容，反之亦然。

## 0x03 Static Route

为了让内网设备能够直接访问对方的内网设备，我们需要在路由器上添加静态路由。

由于已经打开了`IP转发`，所以`Cat`和`Neko`已经可以当作网关来使用了。

去`OpenWrt` -> `网络` -> `静态路由`，添加一条静态路由。

- `Cat`端的路由器

|接口|目标|网关|跃点数|表|
|:---:|:---:|:---:|:---:|:---:|
|lan: [桥接: "br-lan"]|192.168.124.0/24|192.168.1.233|0|main|

- `Neko`端的路由器

|接口|目标|网关|跃点数|表|
|:---:|:---:|:---:|:---:|:---:|
|lan: [桥接: "br-lan"]|192.168.1.0/24|192.168.124.233|0|main|

然后，你就可以在内网里面直接访问对方的内网设备了。

## 0x04 其他的

其实比起用`DDNS`,我在想是不是能直接用`mDNS`让两台设备在内网里面互相发现（

两边的路由器稍微配置一下`avahi-daemon`，估计可以实现。

然后，通过类似的方法你也可以把家里的网段打通。
