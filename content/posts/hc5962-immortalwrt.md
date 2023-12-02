---
title: "HiWiFi HC5962 ImmortalWrt 固件"
date: 2023-10-13T21:46:26+08:00
draft: false
keywords: ["openwrt", "HC5962", "HiWiFi", "ImmortalWrt"]
tags: ["Router", "Resources"]
summary: "博主自己编译的固件，如果你有这台路由器，可以试试。"
---

## 0x00 前言

博主有一个`HiWiFi HC5962`，其实也就是当年的`极路由4Pro 增强版`。

这台路由是好几年前买来尝鲜的，因为听朋友说插件生态很有趣。当年确实也是这样，可惜就可惜在`极路由`后来倒闭了。
他们的官网也变成了一些奇奇怪怪的东西。
据说是因为他们做`挖矿路由器`，做的亏本跑路了。反正我觉得挺可笑的，路由器就应该做他应该做的事情，而不是挖矿。
我也不认为挖矿是好的，你要是有这些算力，建议送给我，而不是浪费在挖矿上。

如果你现在手上有一台原厂的极路由，然后你也想装一些别的固件，比如`OpenWRT`，
你可以去 **[http://www.hiwifi.wtf](http://www.hiwifi.wtf)** 来开放SSH，具体操作你可以`Google`一下，
网上应该还挺多的。进去以后用`mtd-write`写一下`breed`，然后就可以刷你喜欢的固件了。

本文只是提供博主自用的固件，如果帮到你了我很荣幸。

## 0x01 东西

新版本的`OpenWrt`支持了`HC5962`的`gmac1`，硬件流量分载性能十分不错。在我这多播8个`macvlan`能达到峰值`800Mbps`，还不错。

我用了 **[这个](https://github.com/immortalwrt/immortalwrt)** 仓库，里面的`README.md`有十分详细的编译教程，如果你想要自己编译，
你可以去看看。

编译这个固件主要是为了内置`syncdial`和一些多播必要的东西。如果你不需要这些，也可以选择`ImmortalWrt`预编译的固件，它在 **[这里](https://firmware-selector.immortalwrt.org/)** ，其余效果是一样的。

这是我的 **[.config](https://nvme0n1p1.brain0.dev/d/AlidriveShare/Resources/HC5962/.config)** ，
你有兴趣的话可以用这个来编译。

 **[这些](https://nvme0n1p1.brain0.dev/d/AlidriveShare/Resources/HC5962/immortalwrt-ramips-mt7621-hiwifi_hc5962.tar.xz)** 是我编译好的固件，如果你要用的话，它包含这些内容：

```bash
.
├── config.buildinfo
├── feeds.buildinfo
├── immortalwrt-ramips-mt7621-hiwifi_hc5962-initramfs-kernel.bin
├── immortalwrt-ramips-mt7621-hiwifi_hc5962.manifest
├── immortalwrt-ramips-mt7621-hiwifi_hc5962-squashfs-factory.bin
├── immortalwrt-ramips-mt7621-hiwifi_hc5962-squashfs-sysupgrade.bin
├── packages
│   ├── base-files_1687-r27102-3810187d20_mipsel_24kc.ipk
│   ├── block-mount_2023-02-28-bfe882d5-1_mipsel_24kc.ipk
│   ├── dropbear_2022.82-5_mipsel_24kc.ipk
│   ├── fstools_2023-02-28-bfe882d5-1_mipsel_24kc.ipk
│   ├── fwtool_2019-11-12-8f7fe925-1_mipsel_24kc.ipk
│   ├── index.json
│   ├── ip6tables-nft_1.8.8-1_mipsel_24kc.ipk
│   ├── iptables-mod-conntrack-extra_1.8.8-1_mipsel_24kc.ipk
│   ├── iptables-mod-ipopt_1.8.8-1_mipsel_24kc.ipk
│   ├── iptables-nft_1.8.8-1_mipsel_24kc.ipk
│   ├── kernel_5.15.132-1-647e0b8623d500de5191278d83c5ad73_mipsel_24kc.ipk
│   ├── kmod-asn1-decoder_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-cfg80211_5.15.132+6.1.24-3_mipsel_24kc.ipk
│   ├── kmod-crypto-acompress_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-crypto-aead_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-crypto-arc4_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-crypto-authenc_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-crypto-ccm_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-crypto-cmac_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-crypto-crc32c_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-crypto-ctr_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-cryptodev_5.15.132+1.12-ramips-1_mipsel_24kc.ipk
│   ├── kmod-crypto-ecb_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-crypto-gcm_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-crypto-gf128_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-crypto-ghash_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-crypto-hash_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-crypto-hmac_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-crypto-lib-chacha20_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-crypto-lib-chacha20poly1305_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-crypto-lib-curve25519_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-crypto-lib-poly1305_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-crypto-manager_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-crypto-null_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-crypto-rng_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-crypto-seqiv_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-crypto-sha1_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-crypto-sha512_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-crypto-user_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-gpio-button-hotplug_5.15.132-3_mipsel_24kc.ipk
│   ├── kmod-ip6tables_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-ipt-conntrack_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-ipt-conntrack-extra_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-ipt-core_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-ipt-ipopt_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-ipt-ipset_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-iptunnel4_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-iptunnel_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-leds-gpio_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-lib-crc32c_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-lib-crc-ccitt_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-lib-lzo_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-lib-textsearch_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-mac80211_5.15.132+6.1.24-3_mipsel_24kc.ipk
│   ├── kmod-macvlan_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-mppe_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-mt7603_5.15.132+2023-08-14-b14c2351-1_mipsel_24kc.ipk
│   ├── kmod-mt76-core_5.15.132+2023-08-14-b14c2351-1_mipsel_24kc.ipk
│   ├── kmod-mt76x02-common_5.15.132+2023-08-14-b14c2351-1_mipsel_24kc.ipk
│   ├── kmod-mt76x2_5.15.132+2023-08-14-b14c2351-1_mipsel_24kc.ipk
│   ├── kmod-mt76x2-common_5.15.132+2023-08-14-b14c2351-1_mipsel_24kc.ipk
│   ├── kmod-nf-conncount_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-nf-conntrack_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-nf-conntrack6_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-nf-conntrack-netlink_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-nf-flow_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-nf-ipt_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-nf-ipt6_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-nf-log_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-nf-log6_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-nf-nat_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-nf-nathelper_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-nf-nathelper-extra_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-nfnetlink_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-nf-reject_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-nf-reject6_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-nft-compat_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-nft-core_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-nft-fib_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-nft-fullcone_5.15.132+2023-01-10-95ad79bc-1_mipsel_24kc.ipk
│   ├── kmod-nft-nat_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-nft-offload_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-nls-base_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-ppp_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-pppoe_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-pppox_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-sit_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-slhc_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-tun_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-udptunnel4_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-udptunnel6_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-usb3_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-usb-core_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-usb-xhci-hcd_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-usb-xhci-mtk_5.15.132-1_mipsel_24kc.ipk
│   ├── kmod-wireguard_5.15.132-1_mipsel_24kc.ipk
│   ├── libatomic1_12.3.0-4_mipsel_24kc.ipk
│   ├── libc_1.2.4-4_mipsel_24kc.ipk
│   ├── libgcc1_12.3.0-4_mipsel_24kc.ipk
│   ├── libiptext0_1.8.8-1_mipsel_24kc.ipk
│   ├── libiptext6-0_1.8.8-1_mipsel_24kc.ipk
│   ├── libiptext-nft0_1.8.8-1_mipsel_24kc.ipk
│   ├── libpthread_1.2.4-4_mipsel_24kc.ipk
│   ├── librt_1.2.4-4_mipsel_24kc.ipk
│   ├── libstdcpp6_12.3.0-4_mipsel_24kc.ipk
│   ├── libxtables12_1.8.8-1_mipsel_24kc.ipk
│   ├── mtd_26_mipsel_24kc.ipk
│   ├── Packages
│   ├── Packages.gz
│   ├── Packages.manifest
│   ├── Packages.sig
│   ├── ubi-utils_2.1.5-1_mipsel_24kc.ipk
│   └── xtables-nft_1.8.8-1_mipsel_24kc.ipk
├── profiles.json
├── sha256sums
└── version.buildinfo

2 directories, 122 files
```
