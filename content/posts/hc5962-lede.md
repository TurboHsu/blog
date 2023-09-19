---
title: "HiWiFi HC5962 OpenWRT/LEDE 固件"
date: 2023-09-19T16:14:26+08:00
draft: false
---

## 0x00 前言

博主有一个`HiWiFi HC5962`，其实也就是当年的`极路由4Pro 增强版`。

这台路由是好几年前买来尝鲜的，因为听朋友说插件生态很有趣。当年确实也是这样，可惜就可惜在`极路由`后来倒闭了。
他们的官网也变成了一些奇奇怪怪的东西。
据说是因为他们做`挖矿路由器`，做的亏本跑路了。反正我觉得挺可笑的，路由器就应该做他应该做的事情，而不是挖矿。
我也不认为挖矿是好的，你要是有这些算力，建议送给我，而不是浪费在挖矿上。

如果你现在手上有一台原厂的极路由，然后你也想装一些别的固件，比如`OpenWRT/LEDE`，
你可以去 **[http://www.hiwifi.wtf](http://www.hiwifi.wtf)** 来开放SSH，具体操作你可以`Google`一下，
网上应该还挺多的。进去以后用`mtd-write`写一下`breed`，然后就可以刷你喜欢的固件了。

本文只是提供博主自用的固件，如果帮到你了我很荣幸。

## 0x01 东西

我用了 **[这个](https://github.com/coolsnowwolf/lede)** 仓库，里面的`README.md`有十分详细的编译教程，如果你想要自己编译，
你可以去看看。

这是我的 **[.config](https://nvme0n1p1.brain0.dev/d/AlidriveShare/Resources/HC5962/.config)** ，
你有兴趣的话可以用这个来编译。

 **[这些](https://nvme0n1p1.brain0.dev/d/AlidriveShare/Resources/HC5962/openwrt-ramips-mt7621-hiwifi_hc5962.tar.xz)** 是我编译好的固件，如果你要用的话，它包含这些内容：

```bash
.
├── config.buildinfo
├── feeds.buildinfo
├── openwrt-ramips-mt7621-hiwifi_hc5962-initramfs-kernel.bin
├── openwrt-ramips-mt7621-hiwifi_hc5962.manifest
├── openwrt-ramips-mt7621-hiwifi_hc5962-squashfs-factory.bin
├── openwrt-ramips-mt7621-hiwifi_hc5962-squashfs-sysupgrade.bin
├── packages
│   ├── base-files_68-r6176-f56df1631_mipsel_24kc.ipk
│   ├── block-mount_2023-01-22-1ea5855e-1_mipsel_24kc.ipk
│   ├── dropbear_2022.82-4_mipsel_24kc.ipk
│   ├── fstools_2023-01-22-1ea5855e-1_mipsel_24kc.ipk
│   ├── fwtool_2019-11-12-8f7fe925-1_mipsel_24kc.ipk
│   ├── iptables_1.8.7-2_mipsel_24kc.ipk
│   ├── iptables-mod-conntrack-extra_1.8.7-2_mipsel_24kc.ipk
│   ├── iptables-mod-extra_1.8.7-2_mipsel_24kc.ipk
│   ├── iptables-mod-ipopt_1.8.7-2_mipsel_24kc.ipk
│   ├── iptables-mod-tproxy_1.8.7-2_mipsel_24kc.ipk
│   ├── iwinfo_2023-05-17-c9f5c3f7-1_mipsel_24kc.ipk
│   ├── kernel_5.4.255-1-823a2202b20c1ce81300b00809ae8347_mipsel_24kc.ipk
│   ├── kmod-asn1-decoder_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-cfg80211_5.4.255+6.1.24-3_mipsel_24kc.ipk
│   ├── kmod-crypto-acompress_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-crypto-aead_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-crypto-arc4_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-crypto-authenc_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-crypto-ccm_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-crypto-cmac_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-crypto-ctr_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-crypto-des_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-cryptodev_5.4.255+1.12-ramips-1_mipsel_24kc.ipk
│   ├── kmod-crypto-ecb_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-crypto-gcm_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-crypto-gf128_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-crypto-ghash_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-crypto-hash_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-crypto-hmac_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-crypto-hw-eip93_5.4.255+ca08387b-1_mipsel_24kc.ipk
│   ├── kmod-crypto-manager_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-crypto-md5_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-crypto-null_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-crypto-rng_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-crypto-seqiv_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-crypto-sha1_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-crypto-sha256_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-crypto-user_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-gpio-button-hotplug_5.4.255-3_mipsel_24kc.ipk
│   ├── kmod-ipt-conntrack_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-ipt-conntrack-extra_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-ipt-core_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-ipt-extra_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-ipt-fullconenat_5.4.255+2022-02-13-108a36cb-10_mipsel_24kc.ipk
│   ├── kmod-ipt-ipopt_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-ipt-ipset_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-ipt-nat_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-ipt-offload_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-ipt-raw_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-ipt-tproxy_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-leds-gpio_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-lib-crc-ccitt_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-lib-lzo_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-lib-textsearch_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-mac80211_5.4.255+6.1.24-3_mipsel_24kc.ipk
│   ├── kmod-macvlan_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-mppe_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-mt7603_5.4.255+2022-12-22-5b509e80-1_mipsel_24kc.ipk
│   ├── kmod-mt76-core_5.4.255+2022-12-22-5b509e80-1_mipsel_24kc.ipk
│   ├── kmod-mt76x02-common_5.4.255+2022-12-22-5b509e80-1_mipsel_24kc.ipk
│   ├── kmod-mt76x2_5.4.255+2022-12-22-5b509e80-1_mipsel_24kc.ipk
│   ├── kmod-mt76x2-common_5.4.255+2022-12-22-5b509e80-1_mipsel_24kc.ipk
│   ├── kmod-mtk-hnat_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-nf-conntrack_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-nf-conntrack6_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-nf-conntrack-netlink_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-nf-flow_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-nf-ipt_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-nf-log_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-nf-nat_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-nf-nathelper_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-nf-nathelper-extra_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-nfnetlink_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-nf-reject_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-nf-tproxy_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-nls-base_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-ppp_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-pppoe_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-pppox_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-slhc_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-tcp-bbr_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-tun_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-usb3_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-usb-core_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-usb-xhci-hcd_5.4.255-1_mipsel_24kc.ipk
│   ├── kmod-usb-xhci-mtk_5.4.255-1_mipsel_24kc.ipk
│   ├── libatomic1_8.4.0-4_mipsel_24kc.ipk
│   ├── libc_1.2.3-4_mipsel_24kc.ipk
│   ├── libgcc1_8.4.0-4_mipsel_24kc.ipk
│   ├── libip4tc2_1.8.7-2_mipsel_24kc.ipk
│   ├── libip6tc2_1.8.7-2_mipsel_24kc.ipk
│   ├── libiwinfo20230121_2023-05-17-c9f5c3f7-1_mipsel_24kc.ipk
│   ├── libiwinfo-data_2023-05-17-c9f5c3f7-1_mipsel_24kc.ipk
│   ├── libiwinfo-lua_2023-05-17-c9f5c3f7-1_mipsel_24kc.ipk
│   ├── libpthread_1.2.3-4_mipsel_24kc.ipk
│   ├── librt_1.2.3-4_mipsel_24kc.ipk
│   ├── libxtables12_1.8.7-2_mipsel_24kc.ipk
│   ├── mtd_26_mipsel_24kc.ipk
│   ├── Packages
│   ├── Packages.gz
│   ├── Packages.manifest
│   ├── Packages.sig
│   └── ubi-utils_2.1.5-1_mipsel_24kc.ipk
├── profiles.json
├── sha256sums
└── version.buildinfo

2 directories, 112 files
```
