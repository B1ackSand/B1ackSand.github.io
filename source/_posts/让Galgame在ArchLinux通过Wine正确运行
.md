---
title: 让Galgame在ArchLinux通过Wine正确运行
date: 2021-08-25 03:23:02
tags: [Galgame, Wine, Linux]
categories: Linux
cover: https://z3.ax1x.com/2021/08/25/hESU4H.png
sticky: 2
copyright_author: B1ackSand
copyright_author_href: https://github.com/B1ackSand
copyright_url: https://blacksand.top/2021/08/25/让Galgame在ArchLinux通过Wine正确运行
copyright_info: 此文章版权归B1ackSand所有，如有转载，请注明来自原作者

---



仅是一份运行记录，以作备忘。若有帮助，荣幸至极。

由于难以忍受我的电脑在 Windows10 下各种奇葩的性能问题以及强制更新，我在六月初的时候投向了 Arch 的怀抱，并开始作为主力日常使用。而 Windows10，我单独分配了一个约 60G 的空间以作备用：如游玩 3A，带反作弊的游戏，视频渲染等，这些工作在 Windows 下会比 Linux 更加优秀，简单。

但说句不好听的，~~Windows 的强制更新就像 NTR 了我的电脑。~~(滑稽)

本文基于 ArchLinux 编写，大致方向可适用于 Arch 衍生的发行版，Ubuntu 等…

# 前言

大部分 galgame 日文原程序在 Linux 下通过 [Wine](https://www.winehq.org/) 调试后再运行是没有问题的，因为不会涉及到太多东西，DLL 大多数都封装好在游戏目录中，跑起来和 Windows 下无异。但是经过汉化修改过的 Gal 程序，有些会完全无法运行（对，纯属运气问题），因为每个汉化组的汉化手段不一致，可能在 Windows 下能正常运行的，在 Wine 下运行会无反应或花式报错。

即使汉化无法运行，还可以通过使用虚拟机（如 KVM，VmWare）的方式使得程序在 Windows 下运行，但相应在性能上，至少我的设备用虚拟机鼠标拖动有小许延迟和撕裂，galgame 大部分时候画面变化不明显，所以体验尚可。

当然 Steam 上也有原生支持 Linux 的 gal，也有让 Valve 列入白名单的 gal，例如[命运石之门](https://www.protondb.com/app/412830)，这种 gal 几乎可以开箱即用，体验极佳。

如果让我排一个选择优先级，我认为是：

1. Native Game or Whitelisted game
2. Steam Play (Proton)
3. Wine
4. Virtual Machine running Windows
5. Windows

**如果 Steam Play 和 Wine 折腾之后，仍然无法让程序运行，请马上转向使用虚拟机或切换至 Windows，我是玩游戏的而不是被游戏玩的（笑）。**

本文已默认拥有的一个相对完备的系统，已配置好当前系统的显卡驱动、声音驱动还有字体等。

# 个人硬件配置

[![image-20210824171230339](https://z3.ax1x.com/2021/08/25/hAzATe.png)](https://z3.ax1x.com/2021/08/25/hAzATe.png)

# 什么是 Wine

> [Wine](https://www.winehq.org/) 是在 x86、x86-64 容许类 Unix 操作系统在 X Window System 运行 Microsoft Windows 程序的软件。另外，Wine 也提供程序运行库（Winelib）来帮助计算机程序设计师将 Windows 程序移植到类 Unix 系统。
>
> Wine 通过提供一个兼容层来将 Windows 的系统调用转换成与 POSIX 标准的系统调用。它还提供了 Windows 系统运行库的替代品和一些系统组件的替代品。为了避免著作权问题，Wine 主要使用黑箱测试逆向工程来编写。
>
> Wine 最早是 “Windows Emulator”，即 Windows 模拟器的缩写，但 Wine 现在为 “Wine Is Not an Emulator” 的递归缩写，即 Wine 不是模拟器。Wine 的正确名称是 “Wine”，而不是全大写或全小写。
>
> —— 摘自[维基百科简体中文](https://zh.wikipedia.org/wiki/Wine)

# 安装 Wine 并尝试初始化

不同的发行版请参阅 [Wine](https://www.winehq.org/) 官网，或查询其他资料以执行安装。

Wine 需通过开启 [multilib](https://wiki.archlinux.org/title/Multilib) 仓库，来安装 [wine](https://archlinux.org/packages/?name=wine)(稳定版本)，或 [wine-staging](https://archlinux.org/packages/?name=wine-staging)(测试版本)。[Wine-Staging](https://www.wine-staging.com/) 其中包含尚未纳入稳定或开发分支的 bug 修复和功能，加入新句柄的同时，也意味着可能有更多的不稳定因素出现。

WineWiki-FAQ 可查看有关更多信息：[查看链接](https://wiki.winehq.org/FAQ)

## 安装 Wine 及其依赖

安装 Wine 需要启用 `multilib` 存储库，请取消 `/etc/pacman.conf` 中被注释处理的 `[multilib]` 部分：

```
SHELL# 在/etc/pacman.conf文件中：

[multilib]
Include = /etc/pacman.d/mirrorlist
```

随后使用`:wq` 退出保存，刷新 pacman 数据库：

```
SHELL
sudo pacman -Syu
```

刷新 pacman 数据库后，在终端安装 Wine 或 Wine-staging：

```
SHELL# 稳定版
pacman -S wine

# 测试版
pacman -S wine-staging
```

为了让游戏运行正常，我仍然建议安装 Wine 的所有依赖项，以确保 Wine 工作正常：

```
SHELLsudo pacman -S --needed wine-staging giflib lib32-giflib libpng lib32-libpng libldap lib32-libldap gnutls lib32-gnutls \
mpg123 lib32-mpg123 openal lib32-openal v4l-utils lib32-v4l-utils libpulse lib32-libpulse libgpg-error \
lib32-libgpg-error alsa-plugins lib32-alsa-plugins alsa-lib lib32-alsa-lib libjpeg-turbo lib32-libjpeg-turbo \
sqlite lib32-sqlite libxcomposite lib32-libxcomposite libxinerama lib32-libgcrypt libgcrypt lib32-libxinerama \
ncurses lib32-ncurses opencl-icd-loader lib32-opencl-icd-loader libxslt lib32-libxslt libva lib32-libva gtk3 \
lib32-gtk3 gst-plugins-base-libs lib32-gst-plugins-base-libs vulkan-icd-loader lib32-vulkan-icd-loader
```

> 如使用其他发行版，请参阅 Lutris 的[官方 docs 文档](https://github.com/lutris/docs/blob/master/WineDependencies.md)获取更多信息，该文档包含了 Wine 所需要的所有依赖项。

安装完成后，可以在终端尝试执行 `wine`，Wine 会在 `~` 下生成`.wine` 文件夹，该文件夹相当于一个虚拟的 Windows 系统盘，其中包含有 Windows 相关的运行文件。我们一般称 `~/.wine` 为 一个 WINEPREFIX (个人称 Wine 虚拟盘)，而各个 Wine 虚拟盘之间是互不影响的，就像是不同的 Windows 计算机一样。

> **注意：Wine 并不是沙盒，它可访问你当前部分 Linux 文件系统。使用 Wine 时禁止使用 sudo 权限，否则 Wine 前缀将可访问到系统所有文件，若出现威胁，病毒可能会瘫痪掉你的系统。**

## 改用 32 位 Wine 虚拟盘

> At present there are some significant bugs that prevent many 32 bit applications from working in a 64 bit wineprefix.
>
> 目前有一些严重的[错误](http://bugs.winehq.org/buglist.cgi?keywords=win64&resolution=---)会导致一些 32 位程序在 64 位 Wine 虚拟盘中无法运行。
>
> ——[WineHQ](https://wiki.winehq.org/FAQ#How_do_I_create_a_32_bit_wineprefix_on_a_64_bit_system.3F)

为了稳定运行，更换成 32 位 Wine 虚拟盘是很有必要做的一件事情，且文章下方即将安装的组件（如 d3dx9，wmp10）都只能在 32 位 Wine 虚拟盘中安装成功。

Wine 在初始化时默认生成的是 64 位 Wine 虚拟盘，所以我们需要在终端，输入：

```
SHELL# 删除原有初始化的64位Wine虚拟盘
rm -r ~/.wine
# WINEARCH指定为win32 在 ~/ 中创建一个新的WINEPREFIX
WINEARCH=win32 winecfg
```

此时 `~/.wine` 会成为一个 32 位的 Windows 虚拟盘。此后直接执行 `wine program.exe` 时都会使用该 32 位 Wine 虚拟盘。

你也可以在命令中添加 `WINEPREFIX=path_to_wineprefix` 来指定一个地址创建，如：

```
SHELL
WINEARCH=win32 WINEPREFIX=~/.win32 winecfg
```

但如果使用指定地址方式创建，想要以 win32 方式运行程序时需指定 `WINEPREFIX` 变量：

```
SHELL
WINEPREFIX=~/.win32 wine program.exe
```

**注意：不要使用一个已存在的文件夹作为 WINEPREFIX 地址来创建一个新的 Wine 虚拟盘。Wine 必须要创建文件夹。如上例中`.win32` 文件夹需不存在于目录中。**

## 拓展相关

遇到需要分别使用 Internet Explorer 和.NET 为依赖的 Windows 程序，可安装 [wine-gecko](https://archlinux.org/packages/?name=wine-gecko) 和 [wine-mono](https://archlinux.org/packages/?name=wine-mono) 软件包。安装后可不必为每一个新建 Wine 前缀安装 mono 和 gecko，意味着允许你脱机工作，请根据自身情况选择是否安装。

仅限 Arch：

```
CODE
pacman -S wine-gecko wine-mono
```

如遇其他问题，可参照 ArchWiki 相关栏目：[查看链接](https://wiki.archlinux.org/title/Wine#Configuration)

# Wine 相关运行库配置

本部分参考该 [Blog](https://piglalala.github.io/2021/04/07/如何在linux环境下愉快的推galgame01-从基础配置运行到/)，为方便起见，此处直接搬运过来了，并根据自身实际情况进行了修改。

## winecfg 配置

[winecfg](https://wiki.winehq.org/Winecfg) 是 Wine 的 GUI 配置工具，可以通过在终端运行 `winecfg` 显示下图界面。`winecfg` 将默认打开 `WINEPREFIX=~/.wine` 的 Wine 虚拟盘。

> 在下面的标签页选择函数库，展示的函数库有的选的就加，没有的自己打字填入框中添加，例如 gstreamer。

[![image-20210824160516093](https://z3.ax1x.com/2021/08/25/hAzpS1.png)](https://z3.ax1x.com/2021/08/25/hAzpS1.png)

## winetricks 安装组件

仅使用 `winecfg` 配置，只会模拟一个非常简单的 Windows 系统，拿去跑部分 Galgame 已经没问题了（如《恋 × シンアイ彼女》汉化）。但一般运行游戏需要使用到一些 DX 的运行库，Galgame 这种也不例外。为了让 Wine 能跑更多的程序，我们需要使用 winetricks 这个工具在 Wine 中配置更多 Windows 组件。

```
SHELL# 安装winetricks
pacman -S winetricks
```

安装完成后，在终端运行 `winetricks`，该工具默认指向 `~/.wine` 为虚拟盘进行配置。正常情况下会显示下面的界面。

[![h8FAcd.png](https://z3.ax1x.com/2021/08/29/h8FAcd.png)](https://z3.ax1x.com/2021/08/29/h8FAcd.png)

如没有问题，安装下面常用软件包，这些包安装完之后能够正常运行大部分的 Galgame。

> `winetricks d3dx9 quartz devenum wmp10 gdiplus dotnet40 ffdshow cjkfonts `~~vcrun6~~
>
> **注意**： **vcrun6** 暂[无法安装](https://github.com/Winetricks/winetricks/issues/1804)，安装后会破坏当前使用的 wine 前缀。(至少在 wine-6.15 Staging 中)

## 补充：GStreamer 安装

> 根据 [ArchWiki](https://wiki.archlinux.org/title/GStreamer) 指引，需安装：
>
> ```
> pacman -S GStreamer
> pacman -S gst-plugins-good gst-plugins-ugly
> ```

*我未曾玩过需要 GStreamer 的 Gal，仅就按照上面 [Blog](https://piglalala.github.io/2021/04/07/如何在linux环境下愉快的推galgame01-从基础配置运行到/) 的安装搬运过来。*

> 浏览器前往 https://github.com/Nevcairiel/LAVFilters/releases 下载最新安装包。
> 使用相应的 wine 虚拟盘运行该安装包即可。
>
> 如我想给 `~/.wine` 安装 GStreamer，只需 `wine install_name.exe` 即可。

# 运行策略

## Steam Play 兼容性工具 (Proton)

如果在 Steam 中购买了 Galgame 的正版，通过 [ProtonDB](https://www.protondb.com/) 可以找到相应的方案，快速定位并解决问题，而无需自己用 winetrick 或 winecfg 反复调试。一般通过 Steam 运行的正版游戏兼容性会比单纯的 Wine ~~高到不知那里去了~~ 好太多了。

同样在 Proton 中也有类似 winetricks 的软件包，称为 `protontricks`，可以实现类似 winetricks 的功能。

~~如果连 Steam 都不会装，这部分大概也不用看了罢。~~

### 什么是 Proton

> Proton 是 Valve Software 发布的集成于 Steam Play 中的一个新工具，使得在 Linux 玩 Windows 游戏变得非常简单。在其背后，Proton 整合了其他像 Wine 和 DXVK 这样的流行的工具，使得游戏玩家不必自行进行安装和维护。这极大地免除了用户切换到 Linux 的负担，不需要学习系统底层知识，也不会失去很多心爱的游戏。Proton 仍在起步阶段，因此对不同游戏的支持参差不齐，但 Proton 正在不断迭代中。
>
> ——[ProtonDB_MainPage](https://www.protondb.com/)

### 如何激活 Proton

激活 Proton 非常简单，只需前往 Steam > 设置 > Steam Play ，勾选 **为支持的产品启用 Steam Play** 和 **为所有其他产品启用 Steam Play** 。

点击确定后，Steam 可能会请求是否重新启动 Steam 以更新设置，点击确认重启。

[![image-20210829002320664](https://z3.ax1x.com/2021/08/29/h8Fl9g.png)](https://z3.ax1x.com/2021/08/29/h8Fl9g.png)

### Proton 运行示例

此处我以 Riddle Joker 为例，展示如何根据他人的报告进行调整。

[![image-20210828233429994](https://z3.ax1x.com/2021/08/29/h8F13Q.png)](https://z3.ax1x.com/2021/08/29/h8F13Q.png)

- > 启动参数：使用 Wine D3D
  >
  > use PROTON_USE_WINED3D=1 %command% to get past initial blank screen.

  在库中右键 Riddle Joker—— 属性 —— 通用一栏中，将 `PROTON_USE_WINED3D=1 %command%` 填入启动选项框中。

  [![image-20210829000701667](https://z3.ax1x.com/2021/08/29/h8FJun.png)](https://z3.ax1x.com/2021/08/29/h8FJun.png)

- > 右侧栏：TINKER Proton 5.0-10

  报告此处使用的是 Proton 5.0-10，在库中右键 Riddle Joker—— 属性 —— 兼容性中，将 **强制使用特定 Steam Play 兼容性工具** 打勾，随后在下方的选择栏中选择 **Proton 5.0-10**，这样可单独为游戏指定特地的 Proton 版本运行。如曾经没有用过该 Proton 版本，Steam 在游戏启动时会自动下载更新。

  [![image-20210829001647092](https://z3.ax1x.com/2021/08/29/h8FYBq.png)](https://z3.ax1x.com/2021/08/29/h8FYBq.png)

- > 调整：Protontricks
  >
  > Install wmp11 with protontricks 1277930 wmp11 to get the movies to work

  在终端中输入 `protontricks 1277930 wmp11`，为 Riddle Joker 安装 wmp11，以实现 OP 和 ED 的正常播放。

  此处的 `1277930` 的代号是从右键游戏 —— 属性 —— 更新 ——App ID 查询得到。

  **注意：**

1. **若使用 protontricks 安装组件，请切换至想要使用的 Proton 版本后再执行安装，否则将会安装至选择前的 Proton 版本。**
2. **ProtonDB 默认生成的 Wine 虚拟盘为 64 位，即无法安装 wmp10 等仅支持 32 位的组件，所以此处安装的是 wmp11。**
3. 游戏的 ProtonPREFIX（Proton 虚拟盘）会在`游戏程序安装所在目录/../../compatdata/App_ID` 中。

## PlayonLinux

我非常建议使用 PlayOnLinux 去管理 Wine 虚拟盘，有可能你的机器实装的 Wine 出现路径抽风现象，或更新 Wine 后无法直接在终端使用 `wine galgame.exe` 运行时，诸如等等，此时都可以使用 PlayOnLinux 的新 Wine 虚拟盘或旧版本 Wine 尝试运行这个游戏。

此部分写的较为简练，阅读此部分前，请确保将 [Wine 相关运行库配置](https://blacksand.top/2021/08/25/让Galgame在ArchLinux通过Wine正确运行/#Wine相关运行库配置)部分阅读完成。

### 什么是 PlayOnLinux

> [PlayOnLinux](https://wiki.playonlinux.com/index.php/Home) 是一个免费程序，可帮助在 Linux 上安装、运行和管理 Windows 软件。它还可以管理称为 Wine 虚拟盘，并下载和安装某些 Windows 运行库以使某些软件在 Wine 上能够正常运行。
>
> 也可以使用不同的 Wine 版本创建不同的 Wine 虚拟盘，因为在一个旧版本中能够运行的程序，在较新版本上可能无法运行。

### 如何安装 PlayonLinux

在 ArchLinux 下安装，只需在终端执行：

```
SHELL
sudo pacman -Syu playonlinux
```

如使用其他发行版，请访问 PlayonLinux 官方[下载页面](https://www.playonlinux.com/en/download.html)获取更多信息。

### 管理 Wine 版本

打开程序，在**菜单栏**通过`工具` -> `管理Wine版本`即可打开该界面，此处可以下载管理整个 Wine 所有版本。有时，不同的游戏在 Wine 版本之间的运行性和性能不同。Wine 的更新可能会破坏掉之前可直接运行的情况，所以安装多几个 Wine 版本是必要的。

要安装某个版本的 Wine（32 位或 64 位），只需在左侧选择版本，然后按 “>” 按钮下载并安装它。Wine 安装程序可能会向你询问是否安装 Mono 和 Gecko，根据实际情况选择。

要删除版本，请在右侧选中其中一个未锁定的 Wine 版本，按 “<” 按钮删除。如果该版本被创建了虚拟盘，那么该版本将无法进行删除操作，直到虚拟盘被删除。

[![image-20211008041228776](https://z3.ax1x.com/2021/10/08/59gDFs.png)](https://z3.ax1x.com/2021/10/08/59gDFs.png)

### 创建虚拟盘

要在不安装程序的情况下创建驱动器，只需按主程序的`配置`，点击左下角的`新建`。按照程序指导，选择 32 位或 64 位 Wine 版本安装，并为虚拟盘命名。同样的，Wine 安装程序可能会向你询问是否安装 Mono 和 Gecko，根据实际情况选择。创建完成后，它将出现在虚拟盘列表中。

[![image-20211008043413017](https://z3.ax1x.com/2021/10/08/59grYn.png)](https://z3.ax1x.com/2021/10/08/59grYn.png)

### PlayOnLinux 的 winetricks

在主窗口中，按工具栏中的 “配置”，选中一个 Wine 虚拟盘，将显示此窗口。

[![image-20211008045617557](https://z3.ax1x.com/2021/10/08/59gsWq.png)](https://z3.ax1x.com/2021/10/08/59gsWq.png)

在 “Wine” 选项卡有 8 个按钮，包括启动 Wine 配置程序 (winecfg)、控制面板、注册表编辑器、命令提示符等的按钮。

在 “安装内容” 选项卡中，如同 winetricks 一般，可以在里面配置 windows 组件。可参照之前 [winetricks 安装组件](https://blacksand.top/2021/08/25/让Galgame在ArchLinux通过Wine正确运行/#winetricks安装组件)部分，将所必需组件安装上去。

[![image-20211008045847421](https://z3.ax1x.com/2021/10/08/59g6S0.png)](https://z3.ax1x.com/2021/10/08/59g6S0.png)

如果在此处没有找到你需要安装的组件，则可以转到 “杂项” 选项卡，点击`打开一个Shell`，弹出的窗口中默认直接指向了当前虚拟盘的 WINEPREFIX 位置，此时使用 `winetricks` 命令进行安装，默认将会安装在选中的虚拟盘（例如此处我会为 `4.3` 这个虚拟盘安装组件）。

### 安装自定义程序

通过选中`安装`或左侧栏的`安装一个程序`，打开后方的界面，选中左下角的`安装未在列表中的程序`来安装不在 PlayOnLinux 列表中的程序。

[![image-20211008042238325](https://z3.ax1x.com/2021/10/08/59ggyT.png)](https://z3.ax1x.com/2021/10/08/59ggyT.png)

如果你是第一次安装，你会遇到两个弹窗：一个是给你安装程序时的提示，另一个是不要向 WineHQ 提交错误报告，因为 PlayOnLinux 与他们没有关系。

这里引用一下网图：

[![image-20211008051413425](https://z3.ax1x.com/2021/10/08/59ghTJ.png)](https://z3.ax1x.com/2021/10/08/59ghTJ.png)

由于我们已经新建了我们想要的虚拟盘，则此处我们选择`编辑或更新现有的应用程序`。

[![image-20211008044136938](https://z3.ax1x.com/2021/10/08/59g5k9.png)](https://z3.ax1x.com/2021/10/08/59g5k9.png)

我们此处是新安装游戏，所以需点击左下角的`显示虚拟盘`，然后选中想要程序运行的 Wine 虚拟盘；点击下一步后，跳过`安装前想要做什么？`，3 个选项均不勾选；根据虚拟盘 Wine 是 32 位还是 64 位选择 32bit 或 64bit。

[![image-20211008044235904](https://z3.ax1x.com/2021/10/08/59gIYR.png)](https://z3.ax1x.com/2021/10/08/59gIYR.png)

到达浏览安装文件时，我们浏览选中要启动的游戏主 exe，尝试看看能不能运行，若无法运行则需要尝试其他版本。

[![image-20211008044638693](https://z3.ax1x.com/2021/10/08/59gof1.png)](https://z3.ax1x.com/2021/10/08/59gof1.png)

[![image-20211008044838372](https://z3.ax1x.com/2021/10/08/59g7Sx.png)](https://z3.ax1x.com/2021/10/08/59g7Sx.png)

试运行成功过后，PlayOnLinux 将会弹出一个快捷方式创建窗口，如果在窗口中没有找到主程序，则需要自行选择`浏览`，定位到主程序，重命名，并点击下一步生成，此时快捷方式将会出现在桌面。完成后即可点击取消退出安装。

此后你想从 PlayOnLinux 启动游戏或从桌面启动，均可直接运行游戏，就像双击.exe 可执行文件并且能成功运行一般。

[![image-20211008044956106](https://z3.ax1x.com/2021/10/08/59gHl6.png)](https://z3.ax1x.com/2021/10/08/59gHl6.png)

## Shell 脚本的一个妙用

游戏在汉化过程中，汉化组可能用到了一些其他 API 或者 DLL，抑或因为程序修改、重新打包，会出现游戏程序无法正常运行的问题：

如出现 Fatal error 错误窗口弹出（近月少女的礼仪 2 汉化）；

[![image-20210824135114667](https://z3.ax1x.com/2021/08/25/hAz9Qx.png)](https://z3.ax1x.com/2021/08/25/hAz9Qx.png)[![image-20210824162524351](https://z3.ax1x.com/2021/08/25/hAzkwD.png)](https://z3.ax1x.com/2021/08/25/hAzkwD.png)

或游戏目录会出现类似如此的错误报告（少女理论及其周边 汉化）；

[![image-20210824134017711](https://z3.ax1x.com/2021/08/25/hAzCy6.png)](https://z3.ax1x.com/2021/08/25/hAzCy6.png)

又或者直接游戏，无法打开，仅在后台运行，没有任何反应（LOOPERS 汉化）；

[![image-20210824140456246](https://z3.ax1x.com/2021/08/25/hAzPOK.png)](https://z3.ax1x.com/2021/08/25/hAzPOK.png)

------

此处用到了 Wine 的 Debug 功能，成功原理：

> There are some cases where the bug seems to disappear when `WINEDEBUG` is used with the right channel.
>
> 在某些情况下，当 `WINEDEBUG` 与正确的参数通道一起使用时，BUG 似乎会消失。
>
> ——[WineHQ](https://wiki.winehq.org/FAQ#How_do_I_get_a_debug_trace.3F)

我在终端尝试在 `wine` 命令前添加 `WINEDEBUG=+relay` 后，出现上述问题的游戏开始可以运行，但因为在终端输出大量滚动日志，会占用大量 CPU 使用率，游戏性能会大幅下降。

游戏运行必须需要有 Debug 日志输出才可运行，而如果大量的日志条目输出到文件，会占用非常大的硬盘空间。(实测在一分钟内可产生 1G 的日志文件)

我想做的是需要将这些滚动日志重定向到一个临时文件来存储，并在游戏开启后删除该临时日志文件。

[![image-20210824141036586](https://z3.ax1x.com/2021/08/25/hAzFeO.png)](https://z3.ax1x.com/2021/08/25/hAzFeO.png)

我曾尝试将日志输出重定向至 `/dev/null` ，但这种做法无效，Wine 依旧认为程序无法继续执行。

所以我们可以使用一个简单的 shell 脚本启动程序，先让日志输出 5 秒，让 Wine 认为该程序可继续执行，然后再运行游戏过程中删除临时文件，让重定向无法继续输出日志到临时文件。

代码示例如下：

```
SHELL#!/bin/bash
# run.sh
WINEDEBUG=+relay wine /mnt/mounte/Galgame/loopers/SiglusEngine_cn_210729.exe > /tmp/output.txt 2>&1 &

sleep 5
rm -f /tmp/output.txt
```

此后我们可以使用该 shell 脚本来启动程序。

总而言之，可以尝试添加不同的 DEBUG 参数通道运行游戏，说不定可以解决一些问题，具体通道参数信息请查阅 WineHQ。

~~不管有没有至少先试试再说嘛，说不定就能跑得动了也不用麻烦去开虚拟机。~~

## 字体优化

Wine 在运行当中可能缺少字体，即使通过 `winetricks` 安装了 `cjkfonts` 或 `gdiplus` 解决了字体问题，可选择下面的方案进行进一步优化。

### 使用 Windows 分区中的字体

如果安装了 Windows 分区（即双系统），我建议通过拷贝使用其字体：

```
SHELLmkdir /usr/share/fonts/WindowsFonts
cp /windows/Windows/Fonts/* /usr/share/fonts/WindowsFonts/
chmod 644 /usr/share/fonts/WindowsFonts/*
```

然后重新生成 fontconfig 缓存：

```
SHELL
fc-cache --force
```

最后重启系统。

> 如发现字体反而出现了问题，删除 `fonts` 文件夹中的 `WindowsFonts` 文件夹，重新进行 fontconfig 缓存即可。

### 从 Windows ISO 中提取字体

> 资料来源 [ArchWiki](https://wiki.archlinux.org/title/Microsoft_fonts_(简体中文)#从_Windows_ISO_中提取字体)：
>
> 在 Windows ISO 文件中也可以找到字体。若 ISO 是在网络上下载的，则包含字体的文件格式为 WIM (*Windows Imaging Format*) ；若 ISO 是使用 Windows 介质创建工具创建的，则对应文件格式为 ESD (*Windows Electronic Software Download*)。从 ISO 文件中提取 `sources/install.esd` 或 `sources/install.wim` 文件并从中找到 `Windows/Fonts` 目录。它可以用 *7z* ([p7zip](https://wiki.archlinux.org/title/P7zip)) 或 *wimextract* ([wimlib](https://archlinux.org/packages/?name=wimlib)) 提取。以下是一个使用 *7z* 的示例:

```
SHELL7z e Win10_1709_English_x64.iso sources/install.wim
7z e install.wim 1/Windows/{Fonts/"*".{ttf,ttc},System32/Licenses/neutral/"*"/"*"/license.rtf} -ofonts/
7z e install.wim Windows/{Fonts/"*".{ttf,ttc},System32/Licenses/neutral/"*"/"*"/license.rtf} -ofonts/ # Windows 7
```

> 字体和许可证将位于 `fonts` 目录。获得字体后可参照上面步骤将字体拷贝到相应文件夹，随后重启系统。

若仍无效果，尝试将字体复制到 wine 的 Fonts 文件夹: `~/.wine/drive_c/windows/Fonts`，重启系统，查看是否读取字体。

## 虚拟机相关

> 得闲再写。感觉都可以单独拉出一篇文章来写。

# 表格

下表是根据我的电脑情况而汇总的方案，当然发行版不同可能遇到的问题也不同，使用的运行策略也不同，此表**仅供参考**，不定时更新。

个人电脑使用的是最新 Staging 版本的 Wine，同时我可能顺便测试 4.x 或 5.x 或 6.0.1_stable 的 Wine（PlayonLinux）。

Staging 将缩写为 S，表示为测试版。

上次更新的时间：2021/8/29 00:40

| 游戏名称                 | 使用引擎         | 游戏版本                    | Wine (Proton) 版本   | 策略                                                         | 备注             |
| ------------------------ | ---------------- | --------------------------- | -------------------- | ------------------------------------------------------------ | ---------------- |
| 传达不到的爱恋           | AMUSE CRAFT 自制 | 初回版 (x’moeV1 汉化)       | wine-6.15S           | 直接运行                                                     |                  |
| 天色＊アイルノーツ       | 吉里吉里         | PRO2 汉化组 v2016.02.07     | wine-6.15S           | 直接运行                                                     |                  |
| Riddle Joker             | 吉里吉里         | Steam 正版                  | Proton 5.0-10        | [ProtonDB](https://www.protondb.com/app/1277930)             |                  |
| Senren＊Banka            | 吉里吉里         | Steam 正版                  | Proton 5.0-10        | [ProtonDB](https://www.protondb.com/app/1144400)             |                  |
| Sabbat of the Witch      | 吉里吉里         | Steam 正版                  | Proton 5.0-10        | [ProtonDB](https://www.protondb.com/app/888790)              | x’moe 汉化未测试 |
| 喫茶ステラと死神の蝶     | 吉里吉里         | 弥生月汉化组 天之圣杯汉化组 | –                    | [虚拟机](https://blacksand.top/2021/08/25/让Galgame在ArchLinux通过Wine正确运行/#虚拟机相关) | Wine 无法运行    |
| PARQUET                  | 吉里吉里         | Steam 正版                  | Proton 4.11-13       | [Proton](https://www.protondb.com/app/1662840)               | 需做调整         |
| 月に寄りそう乙女の作法   | QLIE             | KID 汉化 v1.21              | wine-6.15S           | 直接运行                                                     |                  |
| 乙女理論とその周辺       | QLIE             | KID 汉化 v1.0               | wine-4.0 wine-6.15S  | [Shell](https://blacksand.top/2021/08/25/让Galgame在ArchLinux通过Wine正确运行/#Shell脚本的一个妙用) |                  |
| 乙女理論とその後の周辺   | QLIE             | KID 汉化 v1.0               | wine-4.10            | 直接运行                                                     |                  |
| 月に寄りそう乙女の作法 2 | QLIE             | KID 汉化 v1.0               | wine-6.15S           | [Shell](https://blacksand.top/2021/08/25/让Galgame在ArchLinux通过Wine正确运行/#Shell脚本的一个妙用) |                  |
| Clover Day’s Plus        | 吉里吉里         | 星冈四叶草调养中心          | wine-4.3S wine-6.0.1 | 直接运行                                                     |                  |
| オトメ＊ドメイン         | 吉里吉里         | ~~黙示 v1.2~~               | wine-4.3S wine-6.15S | 直接运行                                                     |                  |
| LOOPERS                  | SiglusEngine     | 绿茶汉化组 v1.0             | wine-6.15S           | [Shell](https://blacksand.top/2021/08/25/让Galgame在ArchLinux通过Wine正确运行/#Shell脚本的一个妙用) | 运行性能一般     |
| ハミダシクリエイティブ   | Artemis          | 白井木学院汉化组 v1.0       | wine-6.0.1           | 直接运行                                                     | 无法播放影片     |

# 相关链接

> 本文由 typora 编写 [*typora* 官网 - *Typora* — a markdown editor, markdown reader](https://www.typora.io/)

- [「游戏 / 科技」如何在 Linux 环境下愉快的推 Galgame—— 从基础配置运行到解决 OP 视频播放 / 没声音问题 | piglalala’s Blog](https://piglalala.github.io/2021/04/07/如何在linux环境下愉快的推galgame01-从基础配置运行到/)
- [在 Linux 下跑 galgame | Moonsekai’s Blog](https://moonsekai.xyz/blog/Fun/在-Linux-下跑-galgame.html)
- [Wine - ArchWiki](https://wiki.archlinux.org/title/wine)
- [请问目前为止已知的 GALGAME 制作引擎有什么？ - 知乎](https://www.zhihu.com/question/22981630?sort=created)
- [FAQ - WineHQ Wiki](https://wiki.winehq.org/FAQ)
- [ProtonDB | Gaming reports for Linux using Proton and Steam Play](https://www.protondb.com/)
- [PlayOnLinux For Easier Use Of Wine - LinuxAndUbuntu](https://www.linuxandubuntu.com/home/playonlinux-for-easier-use-of-wine)
