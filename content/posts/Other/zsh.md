---
title: "Windows oh-my-zsh 命令行出现乱码的原因及解决办法"
date: 2021-10-16T10:42:38+08:00
categories: ["杂项"]
tags: ["zsh","oh my zsh","Windows"]
# description: "Desc Text."

# weight: 1

TocOpen: false
draft: false

hideSummary: false
hidemeta: false
disableShare: false

comments: true
canonicalURL: "https://www.niuwx.cn/"

cover:
    image: "<image path/url>"
---

本文以Oh-my-zsh为例，Oh-my-post同样适用。

<!--more-->

如图，使用Windows Terminal连接服务器使用Zsh命令行时，会出现乱码，导致主题无法正常显示。

![](/zsh_error.png)

### 解决方法

安装[Nerd Fonts](https://www.nerdfonts.com/)，下载喜欢的字体然后右键选择“为所有用户安装”，打开Windows Terminal修改settings.json中的"fontFace"项为Nerd字体即可，注意要更改为其显示的**正式名称**。

![](/zsh_ok.png)

![](/posh_ok.png)

