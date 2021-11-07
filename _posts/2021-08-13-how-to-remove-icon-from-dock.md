---
title: 如何删除启动台里的残留图标
date: 2021-08-13T08:58:30+09:00
author: epona
layout: post
permalink: /how-to-remove-icon-from-dock/
categories:
  - OTHER
tags:
  - mac
---
<blockquote class="wp-block-quote">
  <p>
    本文介绍如何删除 macOS 内启动台的残留图标。主要操作步骤是找到储存的数据库文件，删除相应的数据，最后重启Dock即可。
  </p>
</blockquote>

## 找到对应的数据库文件

首先我们打开 Finder， 然后在顶部菜单栏选择 `前往 -> 前往文件夹`，也可以使用快捷键 `Command + Shift + G`来直接启动。然后我们输入 `/private/var/folders`, 在新打开的文件夹里我们搜索 `com.apple.dock.launchpad`。接着我们就会看到`db`文件夹，里面有一个`db`文件，这个就是我们所需要的数据库文件。

![001.png](/assets/images/001.png)
<br>
<br>
![002.png](/assets/images/002.png)

## 删除不需要的数据

接着我们使用 `TablePlus` 打开这个db文件，当然你也可以直接用命令行或者其他App来打开这个文件，我们找到 `apps` 表然后删除不需要的数据，保存。

![003.png](/assets/images/003.png)

## 重启Dock

当数据删除之后，我们需要重启`Dock`来让他生效。在终端(`terminal.app`)里面我们可以输入下面的命令来重启。

<pre class="wp-block-code"><code>killall Dock
</code></pre>

此时打开启动台我们就会发现残留的图标已经没有了。