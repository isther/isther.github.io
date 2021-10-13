---
title: "Manjaro下双显卡笔记本配置"
date: 2022-01-23T09:33:51+08:00
categories: ["manjaro"]
tags: ["manjaro","kde",linux","nvidia","amd"]
# description: "Desc Text."

# weight: 1

TocOpen: false
draft: false

hideSummary: false
hidemeta: false

disableHLJS: true # to disable highlightjs
disableShare: false

comments: true
canonicalURL: "https://www.niuwx.cn/"
cover:
    image: "/neofetch.png"
---

前段时间换了系统，开始使用Manjaro，但是驱动问题搞得很头疼，在这里记录一下解决双显卡(amd+nvidia)的显示问题。

由于Manjaro自带了`mhwd`，所以下载驱动还是很方便的。但是下载了`video-hybrid-amd-nvidia-prime`之后，外界显示器会时常卡顿，切换到`video-nvidia`之后，笔记本的显示器又亮不了。

需要使用`optimus-manager`。

[官方文档](https://github.com/Askannz/optimus-manager#important--manjaro-kde-users)

安装：`yay -S optimus-manager`

配置：

1. `sudo vim /etc/sddm.conf`，找到`DisplayCommand`和`DisplayStopCommand`，行首加`#`注释。

2. `sudo vim /etc/optimus-manager/optimus-manager.conf`，添加以下内容：

   ```yaml
   [amd]
   DRI=3
   driver=modesetting
   tearfree=
   
   [intel]
   DRI=3
   accel=
   driver=modesetting
   modeset=yes
   tearfree=
   
   [nvidia]
   DPI=96
   PAT=yes
   allow_external_gpus=no
   dynamic_power_management=no
   ignore_abi=no
   modeset=yes
   options=overclocking
   
   [optimus]
   auto_logout=yes
   pci_power_control=no
   pci_remove=no
   pci_reset=no
   startup_auto_battery_mode=integrated
   startup_auto_extpower_mode=nvidia
   startup_mode=nvidia
   switching=none
   ```

3. `sudo vim /etc/default/grub`，在`GRUB_CMDLINE_LINUX_DEFAULT`中增加`nvidia-drm.modeset=1`。

4. `grub-mkconfig && update-grub`



PS:启用外接显示器后若KDE无法使用桌面动画，建议安装`optimus-manager-qt`：`yay -S optimus-manager-qt`。
