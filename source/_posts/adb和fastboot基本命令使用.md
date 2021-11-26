---
title: adb和fastboot基本命令使用
date: 2021-08-25 14:08:47
tags: Android
categories: Android
cover: https://z3.ax1x.com/2021/08/25/hVmTxK.md.jpg
copyright_author: B1ackSand
copyright_author_href: https://github.com/B1ackSand
copyright_url: https://blacksand.top/2021/08/25/adb和fastboot基本命令使用
copyright_info: 此文章版权归B1ackSand所有，如有转载，请注明来自原作者
---



## adb


1. 查看当前连接的设备，连接到PC的Android设备将被会打印到终端：`adb devices`  

2. 将指定的apk文件安装到设备上：`adb install package_name.apk`  

3. 将指定的软件进行卸载：`adb uninstall  <package_name>`  or `adb uninstall –k <package_name>` ， 加上`-k`表示卸载软件，但是保留配置和缓存文件。

4. 登录到Android设备的shell：`adb shell`  

5. 从电脑上发送文件到设备：`adb push <本地路径> <远程路径>`  

6. 从设备上下载文件到电脑：`adb pull <远程路径> <本地路径>`  

7. 显示adb的帮助信息：`adb help`  

   

## fastboot


1. 进入到BootLoader模式：`adb reboot bootloader`  

2. 查看连接的设备：`fastboot devices`  

3. 把当前路径下的system.img写入到system分区：`fastboot flash system system.img`  

4. 写入缓存分区：`fastboot flash cache cache.img`  

5. 写入用户数据分区：`fastboot flash userdata userdata.img`  

6. 把当前的boot.img写入到boot分区，boot分区存放内核和ramdisk：`fastboot flash boot boot.img`  

7. 把当前目录下的recovery.img写入到recovery分区：`fastboot flash recovery recovery.img`  

8. 设备重启到系统：`fastboot reboot`  

