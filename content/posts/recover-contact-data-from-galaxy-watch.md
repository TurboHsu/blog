+++
title = "从Galaxy Watch Active中恢复联系人数据"
date = "2024-02-01T20:55:00+08:00"
author = "TurboHsu"
tags = ["Journal", "Misc", "IoT"]
keywords = ["Galaxy Watch Active", "数据恢复", "联系人", "导出联系人"]
description = "朋友家人的手机消失了，联系人数据并不存在备份，现在唯一还有联系人数据的地方就是他的智能穿戴设备:Galaxy Watch Active。我想了些办法把它们导出了出来。"
showFullContent = false
hideComments = false
+++

## 0x00 谜题

朋友家人的手机消失了，而且并没有找回，但是联系人数据对他来说很重要。他并没有备份手机的数据，联系人也没有云端备份。这就很糟糕了。

看来使用云服务同步通讯录等数据，或者定期备份一下手机数据存储在别的地方是很有必要的。

不过幸运的是，他有在使用智能穿戴设备 (`Galaxy Watch Active`) ，这个设备没有消失，而且它会存储来自手机的通讯录列表。这就很棒，有机会恢复出联系人数据了。

![没有消失的设备](/images/recover-contact-data-from-galaxy-watch/photo_2024-01-30_19-18-53.jpg)

由于数据量比较大 (大约有 1600 个联系人) ，通过手动输入的方式录入十分耗时费力，这时候就得另求他法了。这里记录了我是如何从这个设备中恢复出联系人数据的。

## 0x01 探索

### 调试桥

`Galaxy Watch Active` 使用的系统是 `Tizen`，是一个由三星开发的，基于`Linux`的移动平台操作系统。这块手表里面使用的`Wearable 5.5`。你可以在[他们的官网](https://www.tizen.org/)找到更多详细信息。

很幸运的，这个系统是开源的，并且有完善的开发工具链。我在他们的官网上找到了[Tizen Studio](https://developer.tizen.org/development/tizen-studio/download)，它包含一套在`Tizen`平台上开发原生应用和网页应用的工具集。我下载了`Tizen Studio 5.5 with IDE installer`。

需要注意的是，这套工具集**十分脆弱**，安装路径不能有空格，否则`Studio`就会无法启动。这玩意的最新版本是2023年10月底发布的，也不是什么年久失修的软件，就很怪。我在使用的时候就遇到了这个问题，后来翻到了[这个](https://stackoverflow.com/questions/57762869/tizen-studio-not-opening)才得以解决。还要吐槽一点，至少我没有找到黑暗模式的开关，这很糟糕。

准备好一切之后，你就可以把手表连接到电脑了。你需要在手表上打开调试模式，然后打开`Device Manager`，输入手表的`IP地址`，就可以建立调试桥的连接。

![建立调试桥的连接](/images/recover-contact-data-from-galaxy-watch/Screenshot%202024-02-01%20213212.png)

再提一嘴，手表疑似在一次启动周期内只能连接到一台电脑。我的意思是，如果你手表的调试桥被别的电脑连接过，就无法连接别的了。你需要重新启动手表，然后再连接。奇怪的设计。

### 文件系统

连接到手表之后，可以在`Device Manager`的右侧对文件系统进行操作，右键该设备还能连接到`Shell`。

我在`owner`的家目录中找到了一个文件夹，其中包含`.media.db`、`.widget.db`之类，并且有个没有权限访问的`privacy`目录。

```sh
sh-3.2$ ls -al

ls: cannot access privacy: Permission denied

total 25716
drwxrwxrwx   3 owner system share        4096 Jan 30  14:48 .
drwxr-xr-x   6 owner system share        4096 Jan 14   2018 ..
-rw-r--r--   1 owner users                  0 Mar 14   2022 .complication_provider.db
-rw-r--r--   1 owner users                  0 Mar 14   2022 .complication_provider.db-journal
-rw-r--r--   1 service_fw service_fw  1724416 Aug 12  22:28 .context-app-history.db
-rw-r--r--   1 service_fw service_fw    32768 Jan 30  15:49 .context-app-history.db-shm
-rw-r--r--   1 service_fw service_fw  4136512 Jan 30  15:49 .context-app-history.db-wal
-rw-r--r--   1 owner users              16384 Jan 14   2018 .ime_info.db
-rw-r--r--   1 owner system share        4096 Oct 15   2021 .media.db
-rw-r--r--   1 owner system_share       32768 Jan 30  14:49 .media.db-shm
-rw-r--r--   1 owner system share     2245432 Jan 30  14:49 .media.db-wal
-rw-r--r--   1 multimedia multimedia    49152 Jan 30  14:48 .media_controller.db
-rw-r--r--   1 multimedia multimedia 17936384 Jan 30  15:48 .resourced-heart-default.db
-rw-r--r--   1 owner users              57968 Jan 30  14:48 .resourced-heart-default.db-journal
-rw-r--r--   1 owner users              16384 Jan 14   2018 .sync-manager.db
-rw-r--r--   1 owner users              12824 Jan 14   2018 .sync-manager.db-journal
-rw-r--r--   1 owner users              28672 Mar 21   2022 .widget.db
-rw-r--r--   1 owner users               4616 Mar 21   2022 .widget.db-journal
d?????????   ? ?     ?                      ?             ? privacy
```

我后来找到了一个联系人同步服务的日志（估计是和手机同步用的），里面记录了联系人数据库存储的位置，就在上述那个无权访问的`privacy`目录当中。比较糟糕，不能够直接拿出数据库了。

### API

直接拿估计是没辙，稍微探索了一下可执行文件，没有能够直接读取联系人数据用的。稍微试了一下，也没有提权的办法。然后我就开始翻阅`Tizen`的开发文档。我看的是[这个](https://docs.tizen.org/application/native/api/wearable/5.5/)，这是`Tizen Wearable 5.5`的原生应用开发文档。

然后我找到了[这个](https://docs.tizen.org/application/native/api/wearable/5.5/group__CAPI__SOCIAL__CONTACTS__SVC__MODULE.html)，里面的函数看上去可以读取通讯录的样子。写个应用，安装到表里，然后读取出通讯录，应该可行。

## 0x02 写东西

### 准备环境

打开`Tizen Studio`的`Package Manager`，安装这一些包：在`Main SDK`下，安装`5.5 Wearable`。在`Extension SDK`中，安装`Samsung Certificate Extension`。前者很显然是为该手表开发应用必须的，后者是为了能够给应用签名，否则无法安装到手表上，这个后面会提到。

哦对了，在安装`Studio`和`SDK`的时候，都有一些文件被我电脑上的杀毒软件(是火绒)给干掉了，是一些动态分析器之类的。如果你想要装的话，最好留意一下，给这些被报毒的文件恢复、添加到信任区。我猜这些是没毒的。

### 向世界问好

全部装好之后，就可以向世界问好了。在`Tizen Studio`中新建了一个项目，创建一个使用`Wearable 5.5 SDK`的`Template`项目就可以。由于实现的功能比较简单，这里使用`Native Application`应该就行。我选的模板是`Basic UI`。

![模板项目](/images/recover-contact-data-from-galaxy-watch/Screenshot%202024-02-01%20223211.png)

模板内源码的语言居然是`C`，挺硬核。

创建完项目之后，我就尝试编译、上传这个模板项目了。但是在手表上安装应用需要签名，我在`Tizen Studio`中直接创建的证书却怎么都无法使用。看来三星设备并不接受普通的签名。后来阅读了[这个回答](https://stackoverflow.com/questions/53079325/the-application-installation-on-the-device-has-failed-due-to-a-signature-error)，才知道需要上文提到的`Samsung Certificate Extension`，用这玩意可以创建三星设备信任的证书，给软件签名。

装了那个拓展之后，创建证书的时候就可以创建三星证书了。这需要你登录三星账号，不过貌似没有什么限制，登录就能签名。疑似比苹果那套证书系统开放多了。挺酷的。

![三星证书](/images/recover-contact-data-from-galaxy-watch/Screenshot%202024-02-01%20224246.png)

创建好三星证书之后，再让程序在手表上运行，手表很成功地向世界问好了。

### 拉取数据

通过查阅文档，我发现有一个函数是这样的：

```c
int contacts_vcard_make_from_contact(contacts_record_h contact, char **vcard_stream);
```

这就很好，刚好是我需要的。它接收一个`contacts_record_h`，给出一个`vCard`流。`vCard`是一种比较通用的存储联系人信息的形式，如果你想要了解它的话，可以看看[维基百科是怎么说的](https://en.wikipedia.org/wiki/VCard)。

文档写的还行，函数名直接说明了它做的事情，挺好的。

研究了一番，写了一个函数，是这样的：

```c
static void perform_action()
{
    contacts_connect();

    // Vars
    contacts_list_h list;
    contacts_record_h record;
    int file_index = 0;
    int ret; // for error handling

    ret = contacts_db_get_all_records(_contacts_contact._uri, 0, 0, &list);
    if (ret != 0)
    {
        dlog_print(DLOG_ERROR, "CONTACT_EXTRACTOR", "Get all record failed with code %d", ret);
        return;
    }

    while (contacts_list_get_current_record_p(list, &record) == CONTACTS_ERROR_NONE)
    {
        dlog_print(DLOG_INFO, "CONTACT_EXTRACTOR", "Saving to contact_%d.vcf", file_index);

        char *vcard_str = NULL;
        ret = contacts_vcard_make_from_contact(record, &vcard_str);
        if (ret != CONTACTS_ERROR_NONE)
        {
            dlog_print(DLOG_ERROR, "CONTACT_EXTRACTOR", "Make vCard failed with code %d :(", ret);
        }

        char file_path[256];
        snprintf(file_path, sizeof(file_path), "/opt/usr/home/owner/media/Downloads/contact_%d.vcf", file_index++);

        FILE *file = fopen(file_path, "w");
        if (file == NULL)
        {
            dlog_print(DLOG_ERROR, "CONTACT_EXTRACTOR", "Open file failed! :(");
        }

        fputs(vcard_str, file);
        fclose(file);
        free(vcard_str);

        // Read the next contact
        if (contacts_list_next(list) != CONTACTS_ERROR_NONE)
        {
            break;
        }
    }

    contacts_list_destroy(list, true);
    contacts_disconnect();
}
```

这个函数做的事情基本上就是读取全部的通讯录，然后把他们转换成`vCard`流，然后存到文件里面。

该代码中的目录是`Tizen`系统中，用户的媒体文件存放目录。一开始我尝试把这些文件直接存储到`/tmp`当中，但是不知道为什么失败了，`fopen()`疑似没有办法在`/tmp`中创建文件，不知道什么原因。后来使用了`mediastorage`的权限，能够成功的在用户媒体目录里面写文件，这大概就够了。

我让这个函数放在`static void app_resume(void *data)`这个函数内执行，根据模板内的注解，这样做应该会让我的函数在应用变得可见的时候执行。

在`tizen-manifest.xml`中声明的需要的权限，我要了这些权限：

```text
http://tizen.org/privilege/mediastorage
http://tizen.org/privilege/contact.read
http://tizen.org/privilege/callhistory.read
http://tizen.org/privilege/externalstorage.appdata
http://tizen.org/privilege/externalstorage
```

应该是申请多了，但是最后它工作了，就这样了。这些权限居然是`URL`，直接访问可以得到该权限的详情，挺有趣的设计。

由于该程序并没有申请权限的步骤，因此没有弹出权限申请框。但是可以在`设置` - `许可管理器`中手动允许需要的权限。编译、上传、手动允许权限之后，可以在`Log`中观察到程序正在生成`vCard`文件，不错。

![导出vcf文件](/static/images/recover-contact-data-from-galaxy-watch/Screenshot%202024-01-30%20214702.png)

之后可以通过`tar`打一个包，然后用`Tizen Studio`中的`Device Manager`拉取打包好的数据，这样就好了。

### vCard 文件的题外话

最后在发给朋友之前稍作研究才知道，这种`vCard(.vcf)`文件是可以在一个文件内包含多个通讯录数据的。它大概长这样：

```text
BEGIN:VCARD
VERSION:3.0
N;CHARSET=UTF-8:XXX;XXX;XXX;;
FN;CHARSET=UTF-8:XXX
TEL;TYPE=VOICE;TYPE=CELL;PREF;CHARSET=UTF-8:XXXXXXXXXXX
REV:XXXXXXXXXXXXX
END:VCARD
```

它是有`BEGIN`和`END`标识的，用`cat *.vcf > sum.vcf`生成一个总通讯录文件也能被正确识别。挺酷的。

## 0xFF 后话

意料之外的，整个过程挺顺利的。

如果你需要我的项目文件，它在[这里](https://github.com/TurboHsu/tizen-contact-extractor)。

此外，我造了一个已经签好名的软件包，它在[这里](https://github.com/TurboHsu/tizen-contact-extractor/blob/main/Debug/cc.owow.contactextractor-1.0.0-arm.tpk)。如果你刚好需要，可以直接从`Device Manager`安装这个软件包。
