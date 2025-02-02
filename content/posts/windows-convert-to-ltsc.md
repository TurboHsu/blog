+++
title = "转换普通 Windows 11 到 LTSC 版本"
date = "2025-02-02T21:01:59+08:00"
author = "TurboHsu"
tags = ["Journal", "Windows"]
keywords = ["Windows", "LTSC"]
description = "如果你讨厌年度更新的话"
+++

## 0x00 什么

本来我是`Windows 11 Pro for Workstation`，但是微软强买强卖 `24H2` ，导致我端坐在电脑面前许久，
等待它更新完成。我的天，我很想玩神秘游戏，这太不酷了。

不过`LTSC`版本不会推送年度更新，并且也有安全更新，是一个不错的去路。

## 0x01 前置准备

### LTSC 镜像

你需要一个 `LTSC` 镜像，你可以从 [这里](https://massgrave.dev/windows_ltsc_links) 得到下载链接。

记得下载**对应语言**的`LTSC`镜像，需要和你原来系统的语言一致。

### 修改注册表

微软不允许你直接通过更换密钥来换到`LTSC`版本，它会提示你`SKU`不一致，我也不知道怎么解决这个问题。

但是你可以让安装器以为你是`LTSC`版本，这样就可以 _"升级"_ 到`LTSC`了。

打开 `regedit`， 去到：

```txt
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion
```

然后，将 `EditionID` 更改为 `IoTEnterpriseS`，将 `ProductName` 更改为 `Windows 11 IoT Enterprise LTSC 2024`.

> 这个方法也可以用在别的不兼容的版本互相转换上。比如反过来把`LTSC`变为`Pro for Workstation`。你可以
在 [这里](https://massgrave.dev/hwid#supported-products) 找到对应的 `EditionID` 和 `ProductName`。

## 0x02 转换

挂载你下载的 `LTSC` 镜像，然后运行 `setup.exe`。一路往下，如果你能看到 `保留个人文件和应用`：

![setup.exe screenshot](/images/windows-convert-to-ltsc/image.png)

那你就赢了，放心的点下一步，你应该可以平滑的转换到`LTSC`。

## 0xFF 哎

虽然这个方法没有失败我，但是建议还是做个备份 _（比如给C:\打一个镜像）_ 。

哦，此外，少用`Windows`或许能解决这个问题。
