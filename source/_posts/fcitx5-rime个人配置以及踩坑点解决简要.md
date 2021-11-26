---
title: fcitx5-rime个人配置以及踩坑点解决简要
date: 2021-10-18 14:18:25
tags: [Linux]
categories: Linux
cover: https://i.loli.net/2021/10/18/AGszuCDQZy1H5hp.png
copyright_author: B1ackSand
copyright_author_href: https://github.com/B1ackSand
copyright_url: https://blacksand.top/2021/10/18/fcitx5-rime个人配置以及踩坑点解决简要/
copyright_info: 此文章版权归B1ackSand所有，如有转载，请注明来自原作者
---

# 为什么要这么做?

初心真的非常非常单纯，就是因为有拼音显示，而且很方便打emoji出来，而fcitx5怎么搞都搞不出来。

就是这个样一个东西：

![image-20211018131152913](https://i.loli.net/2021/10/18/vMytx2NfKI8cpVJ.png)

# 入门

```bash
# yay -S fcitx5-rime noto-fonts-emoji
```



[fcitx5-rime](https://archlinux.org/packages/?name=fcitx5-rime)的配置文件位置：`~/.local/share/fcitx5/rime/`

如果没有文件夹的话，需要自己建立

```
$ mkdir ~/.local/share/fcitx5/rime/
```



# 配置

## Clone别人配置好的项目

这里偷懒直接Clone的[wongdean/rime-settings](https://github.com/wongdean/rime-settings)，因为[rime官方](https://github.com/rime/home/wiki/CustomizationGuide)写的文档佛味和仙侠味实在是太浓了，佛到我看不明白：

![image-20211018132602999](https://i.loli.net/2021/10/18/sNl1POCoBq4f35Y.png)

佛振写的rime真的挺好的，但是讲实话，看上面佛振的文档不如看下面这张图的文档：

![image-20211018133127687](https://i.loli.net/2021/10/18/rvqSTu1mwdR5zV3.png)

## 修改配置

要求其实很简单：我习惯在电脑上面用全拼输入；能将手机上养了3年多的Google拼音输入法的词库导入进来；能输入emoji；用 `,` 和 `.` 来翻页选词；广东人不熟悉拼音，有很高的模糊音需求。

主要是修改`default.custom.yaml`，`luna_pinyin.custom.yaml`和`luna_pinyin.extended.dict.yaml`

### default.custom.yaml

Clone下来的配置文件自带有小鹤双拼和全拼，这里最简单的方式就是直接注释掉小鹤双拼的配置文件即可，这样rime也不会对双拼进行编译。

注意要遵循严格的[yaml](https://docs.ansible.com/ansible/latest/reference_appendices/YAMLSyntax.html)格式。

```yaml
schema_list:
  # - schema: double_pinyin_flypy # 小鹤双拼
  - schema: luna_pinyin # 全拼
  # - schema: double_pinyin # 自然码
```



翻页选词的更改在`key_binder/bindings`这里我在冒号后的下一行加了：

```yaml
key_binder/bindings:
  - accept: "comma"
    send: "Page_Up"
    when: "paging"
  - accept: "period"
    send: "Page_Down"
    when: "has_menu"
```

还是喜欢展开出来的yaml样式。



### luna_pinyin.custom.yaml

要开始自定义词库，需要在`patch`下添加`translator/enable_user_dict`

```yaml
patch:
  # 开启自定义词库
  "translator/enable_user_dict": true
```



在模糊拼音`'speller/algebra'`中，我对`z/c/s`和`an/en/in`一概分不清，所以我修改成这样：

```yaml
'speller/algebra':
  - erase/^xx$/                      # 第一行保留
  # 模糊音定義
  # 需要哪組就刪去行首的 # 號，單雙向任選
  - derive/^([zcs])h/$1/             # zh, ch, sh => z, c, s
  - derive/^([zcs])([^h])/$1h$2/     # z, c, s => zh, ch, sh
    
  # 韻母部份
  #- derive/^([bpmf])eng$/$1ong/      # meng = mong, ...
  - derive/([aei])n$/$1ng/            # an => ang, en => eng, in => ing
  - derive/([aei])ng$/$1n/            # ang => an, eng => en, ing => in
```



### luna_pinyin.extended.dict.yaml

重头戏──自定义词库

从其他输入法导入rime需要用到[studyzy/imewlconverter](https://github.com/studyzy/imewlconverter)，中文名为深蓝词库转换。

手机谷歌拼音输入法的词库导出后，需要做一些预处理：将**英文和阿拉伯数字开头**的词语要删除，这些词深蓝是无法识别的，而且会破坏输出得到的词库。

处理完后，我在aur中直接安装了深蓝词库转换的软件包，用ImeWlConverterCmd即可进行转换：

```bash
# 这里是谷歌拼音输入法转到rime，-i:输入的词库类型 词库路径1 词库路径2 词库路径3 -o:输出的词库类型 输出词库路径
# -os:windows/macos/linux 设置相应系统
ImeWlConverterCmd -i:ggpy ./Downloads/user-dictionary.txt -o:rime ./.local/share/fcitx5/rime/luna_pinyin.google.dict.yaml "-os:linux"
```

其他发行版，了解更多相关转换请看深蓝词库转换的[wiki](https://github.com/studyzy/imewlconverter/wiki/CommandLine)。



输出完成后，我们要对`luna_pinyin.google.dict.yaml`文件的第一行做一些字段添加：

```
---
name: luna_pinyin.google # 注意修改这里的name
version: "2021.10.18"
sort: by_wight
use_preset_vocabulary: false
...
```



保存后，对`luna_pinyin.extended.dict.yaml`编辑，添加我们刚才自定义的词库：

```yaml
import_tables:
  - luna_pinyin
  - luna_pinyin.xiandaihanyu
  - luna_pinyin.chengyusuyu
  - luna_pinyin.popular
  - essay
  - luna_pinyin.computer
  - luna_pinyin.kaifa
  - luna_pinyin.poetry
  - luna_pinyin.chat
  - luna_pinyin.place
  - luna_pinyin.shopping
  - luna_pinyin.website
  - luna_pinyin.mingxing
  - zhwiki
  # 这里是新加的
  - luna_pinyin.google
```

右键重新部署输入法，查看是否可以用到自定义词库。



# 坑点

## Arch无法显示emoji表情

1. 安装noto-fonts包

   1. 

   ```
   local/noto-fonts
   local/noto-fonts-cjk
   local/noto-fonts-emoji
   local/noto-fonts-extra
   ```

   安装完成之后用：

   ```bash
   fc-cache -vf
   ```

   刷新字体缓存。

   

2. 修改`/etc/fonts/local.conf`，加入以下内容

   ```xml
   <?xml version='1.0'?>
   <!DOCTYPE fontconfig SYSTEM 'fonts.dtd'>
   <fontconfig>
   	<match target="font">
   		<test name="family" compare="contains">
   			<string>Noto Color Emoji</string>
   		</test>
   		<edit name="hinting" mode="assign">
   			<bool>true</bool>
   		</edit>
   		<edit name="hintstyle" mode="assign">
   			<const>hintslight</const>
   		</edit>
   		<edit name="embeddedbitmap" mode="assign">
   			<bool>true</bool>
   		</edit>
   	</match>
   </fontconfig>
   ```

   修改完成后保存，并用`fc-cache -vf`重新刷新字体缓存，右键输入法重新载入，尝试输入emoji相关表情发现可以在输入法显示且系统也可以显示。



# 链接相关

[*RIME* | 中州韻輸入法引擎](https://rime.im/)

[Fontconfig and Noto Color Emoji](https://flammie.github.io/dotfiles/fontconfig.html)

[studyzy/imewlconverter: 一款开源免费的输入法词库转换程序](https://github.com/studyzy/imewlconverter)

[Fcitx5 - ArchWiki](https://wiki.archlinux.org/title/Fcitx5_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))

[archlinux 建议装noto fonts 包 可以完美适配表情 🙃#43](https://github.com/fkxxyz/rime-cloverpinyin/issues/43)

[[Woshiluo's Notebook]](https://blog.woshiluo.com/1693.html)

