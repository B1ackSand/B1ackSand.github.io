---
title: Linux下使用Lutris安装并游玩低延迟音频osu!Stable相关指北
date: 2021-10-02 18:35:02
tags: [osu, Wine, Linux, Game]
categories: Linux
cover: https://i.loli.net/2021/10/09/a9oj4NRX6F1PleY.jpg
sticky: 3
copyright_author: B1ackSand
copyright_author_href: https://github.com/B1ackSand
copyright_url: https://blacksand.top/2021/10/02/Linux下使用Lutris安装并游玩低延迟音频osu!%20相关指北/
copyright_info: 此文章版权归B1ackSand所有，如有转载，请注明来自原作者
---

适合**了解Wine以及 osu!Stable 人士**的指北，内容简单明了，操作基于Arch：[简要版](www.baidu.com) (WIP)

适合**新人**或**想详细了解所做步骤作用**的指北，内容详尽繁多，有疑难解答：[即本文](https://blacksand.top/2021/10/02/Linux下使用Lutris安装并游玩低延迟音频osu!Stable相关指北) 或 [BiliBili专栏版](https://www.bilibili.com/read/cv11906796?from=search&spm_id_from=333.337.0.0)

如果仅仅想要玩 osu! ，**没有刷PP的想法**，为什么不直接使用原生且延迟极低的 [osu!Lazer](https://github.com/ppy/osu) 呢？



---

# 概况

在大概今年六月初的时候，因为实在受不了 Windows10 下系统更新无法持续禁用和莫名其妙的性能等问题，所以在听到stllok小曲奇是在ArchLinux下进行 osu! 的游玩这件事后，我也萌发了 “干脆我就跑去Linux日用吧，而且听说ArchLinux的日用软件体系基本完备，打大游戏和做渲染视频专门用个干净的 Windows10 跑就好了” 的想法。

这篇文章**本意上**是为了进行一个踩坑的总结，在我什么时候系统突然出问题不得不~~REMAKE~~ REINSTALL时，能够重新回来这篇文章进行配置。其次是给有需求的朋友一个**可能有错误**的指导，因为我用Linux的时长也就半年有多。



**另外，本文中的 “音频延迟” 指的并不是降低 osu! 中因为使用 Wasapi Shared 和 DirecetSound 而导致的“音频延迟”，而是降低因为Wine和Linux使用的音频服务器导致的“音频延迟”，使其手感和延迟接近在Windows下游玩 osu! 的感受。**



注意事项：默认你已熟知Linux简单命令操作；拥有一个已经配置相对完好的Linux系统；拥有足够的耐心，以及时间；本文可能有点罗嗦，本质上是为了让各位明白前因和后果；本文基于ArchLinux进行编写。 

![个人为笔记本](https://z3.ax1x.com/2021/10/03/4LlOfJ.png)

下面这两个视频都是在ArchLinux下进行游玩和回放录制的： 

1. [[osu!]senya - 夕立、君と隠れ処 [Insane] +DT 5.73* 253PPFC](https://www.bilibili.com/video/BV1SB4y1T71K)

2. [[osu!] 打了一年半才打出来的哨戒班主难FC](https://www.bilibili.com/video/BV1K44y1z79M)



---

# 安装



## 什么是Wine和Lutris

> Wine （“Wine Is Not an Emulator” 的首字母缩写）是一个能够在多种 POSIX-compliant 操作系统（诸如 Linux，macOS 及 BSD 等）上运行 Windows 应用的兼容层。 简而言之它是一个逆向出来的干净的小Windows，能够调用Windows下的API进行EXE程序的运行。 

> Lutris 是 Linux 的开源游戏安装平台。它的优点是能够在它的平台中进行游戏的安装和自动配置，而不需要用户自行进行复杂的配置，同时也提供大量可视化自定义操作。简而言之Lutris中含有相关游戏和你当前使用发行版的安装脚本，能够像Windows下的安装一样，开箱即用。



## 安装Lutris

为了确保你能够流畅的进行游戏，请务必在系统中安装好相应显卡驱动以及Wine，这将为Lutris的运行提供环境依赖。

ArchLinux下的Lutris可以从Arch Community Repository获取：

```bash
sudo pacman -S lutris
```



> 尽管 Lutris 提供并使用自己的 Wine 构建，我仍建议您安装 Wine 的所有依赖项以确保工作正常。
>
> 如使用其他发行版，请参阅Lutris的[官方docs文档](https://github.com/lutris/docs/blob/master/WineDependencies.md)获取更多信息。



在ArchLinux下安装Wine以及依赖需要启用`multilib` 存储库，请撤销`/etc/pacman.conf`中的[multilib]注释部分： 

```
/etc/pacman.conf
----------------------------------------------------------------------
[multilib]
Include = /etc/pacman.d/mirrorlist
```

更新你的系统：

```bash
sudo pacman -Syu
```

执行下面的命令：

```bash
sudo pacman -S wine-staging giflib lib32-giflib libpng lib32-libpng libldap lib32-libldap gnutls lib32-gnutls \
mpg123 lib32-mpg123 openal lib32-openal v4l-utils lib32-v4l-utils libpulse lib32-libpulse libgpg-error \
lib32-libgpg-error alsa-plugins lib32-alsa-plugins alsa-lib lib32-alsa-lib libjpeg-turbo lib32-libjpeg-turbo \
sqlite lib32-sqlite libxcomposite lib32-libxcomposite libxinerama lib32-libgcrypt libgcrypt lib32-libxinerama \
ncurses lib32-ncurses opencl-icd-loader lib32-opencl-icd-loader libxslt lib32-libxslt libva lib32-libva gtk3 \
lib32-gtk3 gst-plugins-base-libs lib32-gst-plugins-base-libs vulkan-icd-loader lib32-vulkan-icd-loader
```



## 运行Lutris，安装osu!

安装完成后运行Lutris，第一次运行可能需要一把梯子才可运行下去。运行后选中左侧的Sources->Lutris，然后在上方的搜索框进行搜索“osu!”，安装Windows版本的osu!。鉴于国内糟糕的网络环境，也请注意安装过程中osu!也需要使用到梯子。

安装完成后打开，看看是否能够正常运行osu!，并且测试一下打图是否存在问题。值得一提的是此时你会感觉到音频延迟非常非常大，这个问题将会在下个章节解决。

![Lutris下安装Windows](https://z3.ax1x.com/2021/10/03/4L1s39.png)



## 优化调整

1. 在osu!中务必将帧率设置在无限制或者Optimal，否则会出现较大的输入延迟；

2. 在Lutris中，右键osu!->configure，勾选左下角“Show advanced opinions”，转到“System Opinions“，确保“Disable desktop effects”为开启，以消除当前桌面环境的特效合成器导致的延迟；

3. 在“System Options” 最下面的Enviroment Variavles，点击Add，key值栏填入 vblack_mode ，value值栏填入0；

4. 同样在“System Opinions”中，确保“Reduce Pulseaudio latency”为关闭，否则将会破坏掉Wine音频驱动补丁的延迟修正。



---

# 音频延迟修正

实际上在Linux下运行 osu! 是极为轻松远古的一件事情，我甚至在 osu! 论坛上看到了一篇2009年的相关指南。但重要的是不仅仅是只能运行，还需要符合我们的需求。

在Linux上游玩 osu!有一个显著的问题就是音频延迟，原因简析：

第一是由Linux自带的 pulseaudio 音频服务器为了兼容性，对音频延迟做了很大的牺牲，如果不进行一些调整，音频延迟比Windows下面甚至还要高的多。

第二是来源于Wine配置问题，需要调整音频暂存时间参数尽量低，同时保证 osu! 正常播放音频。

更详细的原因我会在文章尾部的链接放出，接下来文章对延迟调整有两个方案，分别为：

1. [gonX的Wine构建，几乎全自动配置。 (最新，会动态调整参数，开箱即用)](#gonX的音频延迟调整方案)

2. [PooN的Wine补丁，需要全手动配置。 (过时，但完全可用，仅作备用和参考)](#PooN的音频延迟调整方案)

个人强烈推荐使用[gonX的方案](#gonX的音频延迟调整方案 (最新，会动态调整参数，开箱即用))。



---

# gonX的音频延迟调整方案

鉴于PooN的补丁完成于三年前，时间久远，补丁配置较困难，打上补丁后甚至无法使用osu!的非兼容模式播放音频（即只能使用DirectSound播放音频，Wasapi Shared在约一年半前推出，当然没有适配音频接口），偶尔出现游戏崩溃现象，gonX修改构建了一个包含动态参数调整，崩溃修复  ~~Wine Debug警告~~ 的Wine6.14构建包，几乎做到了开箱即用，配置友好。



## 从pulseaudio迁移到pipewire (强烈建议)

> [PipeWire](https://wiki.archlinux.org/title/PipeWire)是一个新的低级多媒体框架。它旨在以最小的延迟提供音频和视频的捕获和回放，并支持基于PulseAudio、JACK、ALSA和GStreamer的应用程序。

通过直接使用pipewire，我们可以越过PooN方案中大量的配置文件调整，获得低延迟音频。

gonX说过如果使用他的补丁，**pipewire + osu!关闭音频兼容模式**是最好的选择，参数会自动调整，对于大多数人已足够使用。如坚持使用pulseaudio或开启[音频兼容模式](#启用osu的音频兼容模式)，则需要手动调整一些参数，但仍比PooN的配置简单得多。



有些发行版预装使用PipeWire为音频服务器，此部分可酌情跳过，了解更多pipewire详情请参阅[ArchWiki](https://wiki.archlinux.org/title/PipeWire)。



如何在 Arch 中从 pulseaudio 迁移至 pipewire ：

```bash
# 移除所有pulseaudio服务
systemctl --user disable --now pulseaudio.socket pulseaudio.service

# 安装pipewire pipewire-pulse pipewire-media-session，包管理会提示你将删除pulseaudio的相关软件包，选择Y
sudo pacman -S pipewire pipewire-pulse pipewire-media-session

# 安装完成后，启动pipewire服务
systemctl --user enable --now pipewire.service pipewire.socket pipewire-media-session.service pipewire-pulse.service pipewire-pulse.socket
```

随后对计算机进行重启操作。



## 下载并安装gonX的Wine构建

我强烈建议通过正常渠道下载gonX的Wine构建，这算是对作者的一种感谢：[Google Drive](https://dwz.mk/eQNbuq)（需梯子）

另外我同样提供有[蓝奏盘](https://blacksand.lanzouw.com/b0b3zo18d)的链接下载，提取密码为:1234 。由于格式限制，从蓝奏盘下载下来的文件需要为文件扩展名添加.zst，否则压缩文件将无法正常读取。



如果你正在使用Arch，则可以使用下面命令：

```bash
# Wine构建将默认安装在/opt/wine-osu下，此处以wine-osu-6.14-1-x86_64.pkg.tar.zst为例：
sudo pacman -U ~/Download/wine-osu-6.14-1-x86_64.pkg.tar.zst
```

+ 安装完成后，打开Lutris->右键osu!->configure，在弹出窗口中勾选左下角的 “show advanced options”，转至“Runner Options”页，在“Wine Version”中选择Custom（自定义），然后在下面一栏点击右边的Browse..，选中`/opt/wine-osu/bin/wine`，重要的是选中wine这个文件，选择完成后保存即可。

![Lutris](https://z3.ax1x.com/2021/10/03/4L1gnx.png)

+ 如果是非Arch用户或想要自定义安装路径，则需要打开存档包，个人建议将存档包中/opt文件下的内容解压到Lutris的Wine构建文件夹中，方便Lutris选择自定义Wine构建。

  Lutris默认的Wine构建存放位置：`~/.local/share/lutris/runners/wine/`

  

  解压完成后，打开Lutris->右键osu!->configure，在弹出窗口中勾选左下角的 “show advanced options”，转至“Runner Options”页，在“Wine Version”中选择你刚刚解压的wine-osu文件夹，选择完成后保存即可。

![Lutris](https://z3.ax1x.com/2021/10/03/4L1RHK.png)

## 设置SAP/SAD（针对pulseaudio用户）

**如果现在正在使用pipewire作为音频服务器，则可以完全无视该部分，默认情况下补丁会自动帮你调整这些参数，此时你可以跳转到下一个部分。**



+ 如果你仍在使用 pulseaudio + 关闭音频兼容模式，此时你需要在Lutris中设置`STAGING_AUDIO_PERIOD`（SAP）。右键osu!->configure，转到“System Opinions“，在最下面的Enviroment Variavles，点击Add，key值栏填入 `STAGING_AUDIO_PERIOD` ，value值栏填入10000，然后按Save进行保存。

  打开osu! 进行测试，如果出现音频炸裂的现象，则回到 SAP 处，每次以10000为单位提高value，直到音频正常。

![img](https://z3.ax1x.com/2021/10/03/4L144e.png)



+ 如果你仅能通过pulseaudio + 开启音频兼容模式游玩，则需要在Lutris中设置`STAGING_AUDIO_DURATION`（SAD），添加参数的方法与SAP一致，但是请确保SAD = 2*SAP ，即此时我应该设置SAP为20000，SAD为40000。

![img](https://z3.ax1x.com/2021/10/03/4L1TgA.png)

## 初始化osu! Wine虚拟盘（即WINEPREFIX），使用winetricks安装运行组件

WINEPREFIX可以说是一个最小的Windows系统，但其中并没有安装组件，所以需要一定的初始化。

```bash
# 初始化WINEPREFIX，其中WINE取gonX解压的路径的bin/wine文件，WINEPREFIX取你想将此最小Windows的虚拟盘放置的地方，WINEARCH取win32代表建立32位的wine
WINEARCH=win32 WINEPREFIX=~/.wineosu WINE=/opt/wine-osu/bin/wine winecfg

# 初始化完成后，关闭winecfg，为该虚拟盘安装组件
WINEPREFIX=~/.wineosu WINE=/opt/wine-osu/bin/wine -f -q winetricks cjkfonts gdiplus dotnet45
```

`dotnet45`将不会完全安装，这是wine6版本后的bug，如持续出现：`01f4:fixme:ole:thread_context_callback_ContextCallback 05037D0C, 01DD7011, 04E9FA94, {d7174f82-36b8-4aa8-800a-e963ab2dfab9}, 2, 00000000` 等类似日志错误，则可以按ctrl+c中断。如无法中断请使用系统监视器等软件包处理ms开头的.exe进程即可。 

为什么不使用`dotnet40`而是`dotnet45`，参阅[疑难解答的最后一项](#疑难解答)。



## 配置Lutris 的Wine Prefix，尝试启动osu!

运行组件安装完成后，打开Lutris->右键osu!->configure->Game Options，在Wine Prefix中重新指向你刚才在初始化时设定的`WINEPREFIX`路径，且确保最下面的 “Prefix architecture” 为32-bit，完成后保存。

![img](https://z3.ax1x.com/2021/10/03/4L17jI.png)

在Lutris中打开osu! ，一般此时已经能够正常运行，字体，音频延迟等问题都已解决。



---

# PooN的音频延迟调整方案

注意：PooN的方案在使用pipewire-pulse（即pipewire的pulseaudio兼容实现组件）时可能会出现无法预知的错误，例如没有声音或出现游戏闪退等问题。因本文在撰写时，默认使用的是pulseaudio作为声音服务器，如果正在使用pipewire，我只建议使用gonX的方案修正延迟。



## 更换Lutris的Wine构建（可选）

更换Wine构建说不定可以解决一些问题。由于之前使用tkg-osu-4.6的Wine构建，在我的电脑上无法让游戏跑到显示器的标准刷新率（144Hz），之后我更换使用其他Wine构建，osu! 就能跑到144了。请根据自身情况酌情更改。

步骤：打开Lutris，在左侧栏目的Runner中移到Wine上，点击左侧的下载箭头按钮。然后在里面对`lutris-osu-1`左边的框打勾，他会自动下载新的运行环境。（在最新的wine构建管理器中仅有`lutris-osu-2`的构建，所以此处请转用`lutris-osu-2`）

![img](https://z3.ax1x.com/2021/10/03/4L1XE8.png)

下载完成后，右键osu!->configure->Runner options，在Wine Version中选择新的`lutris-osu-1-x86_64`。



## 启用osu的音频兼容模式

由于PooN的补丁制作完成于三年前，而当时 osu! 并没有实装Wasapi Shared的音频接口。所以需要启用音频兼容模式，让 osu! 调用旧的 DirectSound 音频接口输出音频，若不启用音频兼容模式，打补丁后则会出现音频炸裂或无声的现象。

![若不启用，打补丁后则会出现音频炸裂或无声的现象](https://z3.ax1x.com/2021/10/03/4L1jUS.png)



## 下载音频驱动补丁（由PooN提供）

PooN在大约三年前测试了osu!在pulseaudio下的延迟并且进行了大量调整工作，使音频延迟最终可以降低到15-20ms，这里只需要将Lutris中使用的定制Wine的音频文件进行替换即可。在终端输入：

```bash
# 如果你更换了构建，则需要注意文件夹路径是否正确
wget -O ~/.local/share/lutris/runners/wine/tkg-osu-4.6-x86_64/lib/wine/winepulse.drv.so https://blog.thepoon.fr/assets/articles/2018-06-16-osuLinuxAudioLatency/32bit/winepulse.drv.so

wget -O ~/.local/share/lutris/runners/wine/tkg-osu-4.6-x86_64/lib64/wine/winepulse.drv.so https://blog.thepoon.fr/assets/articles/2018-06-16-osuLinuxAudioLatency/64bit/winepulse.drv.so
```



## 音频相关配置文件调整

在终端使用root身份输入：

```bash
sudo mkdir -p /etc/pulse/daemon.conf.d/
```

该文件是对PulseAudio进行设置并生成在刚才创建的文件夹中：

```bash
echo "high-priority = yes
nice-level = -15

realtime-scheduling = yes
realtime-priority = 50

resample-method = speex-float-0

default-fragments = 2 # Minimum is 2
default-fragment-size-msec = 2 # You can set this to 1, but that will break OBS audio capture." | sudo tee -a /etc/pulse/daemon.conf.d/10-better-latency.conf
```



---

接下来调整 `/etc/security/limits.conf`，以赋予 PulseAudio 更高的系统优先级：

```bash
sudo vim /etc/security/limits.conf
```

到达该文件的最底部，并添加：

```bash
echo "YOURUSERNAME - nice -20

YOURUSERNAME - rtprio 99" >> /etc/security/limits.conf
```

请注意这里的`YOURUSERNAME`替换成你的用户名，例如我的是blacksand：

![替换用户名进入](https://z3.ax1x.com/2021/10/03/4L1v4g.png)



---

最后，编辑`/etc/pulse/default.pa :`

```bash
sudo vim /etc/pulse/default.pa
```

应该可以找到相似的这一行文字：`load-module module-udev-detect`，在其后面添加 tsched=0 ，即为：

![在45行左右的位置](https://z3.ax1x.com/2021/10/03/4L39vn.png)



---

保存所有操作，并重启你的Linux，从而使新的配置文件得到应用。

**敬请注意：**重启后尝试播放音频，如果出现音频撕裂的现象，请在第一步生成的 `10-better-latency.conf` 中，将 `default-fragment-size-msec` 每次调整增加 1 点，保存并重新启动Linux，直到正常播放音频，保证打开osu!后，音频依旧正常。



在播放音频时在终端使用下面的命令可以查看当前系统音频的延迟：

```bash
pacmd list-sinks | grep 'latency: [1-9]'
```

这里是我的终端显示：

```bash
[blacksand@ArchLinux ~]$ pacmd list-sinks | grep 'latency: [1-9]'
        fixed latency: 11.97 ms
        current latency: 12.30 ms
        fixed latency: 12.34 ms
[blacksand@ArchLinux ~]$ 
```



## Lutris相关参数调整

打开 Lutris ，右击游戏 osu!->configure->System options，在最下面的Enviroment Variavles，点击Add，key值栏填入 `STAGING_AUDIO_DURATION` ，value值栏填入10000，然后按Save进行保存。

打开 osu! 进行测试，如果出现音频播放撕裂卡顿的现象，则退出游戏，并且将上面设置的value值每次增加10000，直到音频播放正常。

此参数的意义为将音频暂存时长由默认的800000调整到你需要的数字，从而降低音频延迟。

![System Options](https://z3.ax1x.com/2021/10/03/4L3FbV.png)

## 启动osu! 并开始刷PP

能够正常到达这步基本上没有什么问题了，音频延迟已经解决，游戏正常运行，性能或许比Windows下还要好一些。



---

下面的一些相关问题是我自己个人的需求而出现的，也有一些是性能和文件路径上的问题，各位有兴趣可继续查看下去，或许可以帮到你。



# 疑难解答

+ 为什么我明明使用的是144Hz或者高刷的显示屏，系统显示选项也设置了相应最高刷新率，但是 osu! 依旧处于60Hz锁死的现象？

  可能你体感上能感觉到osu!确实是在60Hz下面运行的，即使是修改osu!里面的帧率限制到无限制，也没有什么作用。实际是默认使用的定制Wine构建的缘故，解决方案请参阅**PooN的音频延迟调整方案**的[第1步](#更换Lutris的Wine构建（可选）)。

  一般来说重启Lutris后打开osu! ，osu! 将会正常按照屏幕标准最高刷新率进行运行。



+ 我该如何在Linux下使用我的数位板进行游玩？

	在Linux下个人推荐使用`OpenTabletDriver`进行数位板连接。这是一个开源、跨平台、可用户自定义的数位板驱动，类似于Windows下的TabletDriver（但TD已经不更新了），比TD功能更强，支持更新的数位板，开发者对Debian系和ArchLinux的安装进行了简化。安装向导：

	```bash
	#安装向导
	https://github.com/OpenTabletDriver/OpenTabletDriver/wiki/Installation-Guide
	
	#常见问题解决方法（包括使用数位板时多个光标同时出现的问题）
	https://github.com/OpenTabletDriver/OpenTabletDriver/wiki/Linux-FAQ
	```

	![OpenTabletDriver](https://z3.ax1x.com/2021/10/03/4L3EUU.png)

+ osu!的安装路径在哪里？我该如何把下载或已存在的谱面进行加载？

	osu! 的安装路径默认位于：`~/Games/osu/drive_c/osu `，想要加载谱面请将谱面下载到osu! 文件夹下的songs目录中，或者直接氪金买撒泼特，下载完成后进入游戏按F5刷新谱面。

	另外如果你是处于双系统的状态或本身有songs文件夹，可以在Linux下的osu!文件夹中建立对其他设备下osu!文件夹songs的链接，但是前提是，若设备是NTFS格式磁盘，需要使用ntfs-3g和相应用户组权限参数进行挂载，否则会出现无法读写问题。

	![此处我对windows下面的songs和skins挂载而来，不用自己移动过来](https://z3.ax1x.com/2021/10/03/4L3V5F.png)

  

+ osu!里面出现日文，韩文全都是框框的现象？

	是由于Wine或Linux中缺少Windows下的日语，韩语等字体而导致的，下面有两种解决方案。

  1. **为osu!运行的Wine虚拟盘安装cjkfonts组件**

     是不是忘记安装`cjkfonts`和`gdiplus`组件了呢？`cjkfonts`是包含有中日韩亚洲字体的组件包，`gdiplus`能解决箭头文本问题，在终端执行：

     ```bash
     WINE=/opt/wine-osu/bin/wine WINEPREFIX=~/.wineosu winetricks cjkfonts gdiplus
     ```

     一般而言执行该操作后，字体将会正常，如仍有问题请参考方案2。

     

  2. **在Linux的fonts文件夹添加Windows下的字体**

     如果你是双系统，你可以将Windows文件夹下的fonts文件复制到 `/usr/share/fonts/WindowsFonts` 下，并执行`chmod -R 755 /usr/share/fonts/WindowsFonts`，需要使用sudo权限执行。

     命令如下：

     ```bash
     sudo cp -r mnt/mountd/Windows/Fonts /usr/share/fonts/WindowsFonts
     
     sudo chmod -R 755 /usr/share/fonts/WindowsFonts
     
     sudo fc-cache -f
     ```

     如果不是双系统则需要自行从iso中提取字体出来，相对麻烦。我自己是不打韩语，俄语等歌曲的，所以我并没有将所有字体放到WindowsFonts中，而是简单提取了3个有日语的字体使用，这3个字体我打了包放到蓝奏盘上面给大家方便下载：[蓝奏云](https://blacksand.lanzoui.com/iEV1Vqr5ybi)

     ![meiryo.ttc msgothic.ttc msmincho.ttc](https://z3.ax1x.com/2021/10/03/4L3eC4.png)

+ 我想录视频或者直播，甚至想要能看到PP计数器那种，我该如何操作？为什么我的pp计数器无法正确读取到osu! 进程？

  录视频和直播请使用`obs-studio`，功能非常强大的推流和录制软件。obs官方的安装教程地址：https://obsproject.com/wiki/install-instructions#linux

  你也可自行构建：https://obsproject.com/wiki/Install-Instructions#linux-build-directions

  

  注意：在ArchLinux的仓库中，默认安装的obs-studio v27不包含有浏览器源，所以此处请用AUR源的`obs-studio-tytan652`，这位老哥打的包能够自动帮助构建浏览器源和obs-studio。

  ![OBS-studio界面](https://i.loli.net/2021/10/17/QLiraGJu2EkfOb6.png)

  PP计数器请使用`gousmemory`，它是一个轻量的，可配合obs浏览器源插件使用的软件：https://github.com/l3lackShark/gosumemory

  解压后修改目录下的config.ini文件，请指定`Path=[songs文件夹路径]`，将 `Wine=false` 改为 `Wine=true` ，执行时需要sudo权限，否则将无法正常读取wine中osu!的内存信息和歌曲信息。启动后，程序会显示本地服务器链接，输入链接到浏览器可查看各种样式。

  ![img](https://z3.ax1x.com/2021/10/03/4L3QDx.png)

  注意：gosumemory至少需要 osu! 运行的Wine中安装有.net 4.5版本及以上才能正常读取 osu! 进程并工作，而在Wine6.0版本后安装`dotnet45`及以上版本，会报错无法运行安装程序，此时需为你的osu!Wine虚拟盘执行强制安装：

  ```bash
  # 此处以dotnet45为例，安装将不会完全执行，但安装的组件足够为gosumemory提供运行环境
  WINE=/opt/wine-osu/bin/wine WINEPREFIX=~/.wineosu winetricks -q -f dotnet45
  ```

  若使用Lutris内建提供的Wine构建，则不需要上述操作。（因为Lutris提供的Wine构建相对较老，且dotnet45已自动安装）

  

+ 如何对显示屏超频或更改显示屏的gamma？

	[Linux下游玩 osu! 修改Gamma值以及显示器刷新率超频备忘、指北 - BiliBili专栏](https://www.bilibili.com/read/cv12671950)
	
	

+ 什么是音频兼容模式？

   请查看PooN方案中的第二步：[点此处跳转](#启用osu的音频兼容模式)

   

+ 为什么 osu! WinePrefix初始化中使用dotnet45而不是dotnet40？

   部分朋友使用`dotnet40`会有：0148:fixme:path:parse_url failed to parse LSystem.Runtime.Remoting.resources 报错，无法启动 osu! 

   初步推断是ppy升级了 osu! 的.net要求。
   `dotnet45`将不会完全安装，这是wine6版本后的bug，如持续出现：01f4:fixme:ole:thread_context_callback_ContextCallback 05037D0C, 01DD7011, 04E9FA94, {d7174f82-36b8-4aa8-800a-e963ab2dfab9}, 2, 00000000 等类似日志错误，则可以按ctrl+c中断。如无法中断请使用系统监视器等软件包处理ms开头的.exe进程即可。 
   
   

---

# 相关参考链接

+ [Low-latency osu! on Linux -- ThePooN Blog](https://blog.thepoon.fr/osuLinuxAudioLatency)  #有关音频延迟的原因和补丁的实现

+ [Linux lutris low latency osu! [GUIDE] -- reddit](https://dwz.mk/NFrYrq)

+ [Ultimate guide to low-latency osu! on Linux (rev.12) -- osu! forum](https://osu.ppy.sh/community/forums/topics/367783)

+ [FS66008 - [obs-studio] Add support for the official browser plugin](https://bugs.archlinux.org/task/66008)

+ [User:Katoumegumi - Arch Wiki](https://wiki.archlinux.org/title/User:Katoumegumi)




---

# 感谢和其他

感谢stllok小曲奇的帮助，我在群里问问题的时候都很热情的回答了我qwq，也是他我才开始真的想用ArchLinux日用；

感谢ArchLinux的这个Wiki平台，详尽的信息让我走了很少弯路；

非常感谢PooN的补丁、文章以及PooN的discord帖子，让在Linux下低延迟进行osu!的游玩成为了可能。



如果文章有什么问题的话请提出，我会进行修正。后面的相关问题什么时候我想到有比较重要的会进行补充，有问题也可以问一下，或许我可能会帮到你。毕竟我做什么事情都会踩很多坑的人（悲），而且我只是站在巨人的肩膀上（谦虚义）才写出来的这篇文章。

