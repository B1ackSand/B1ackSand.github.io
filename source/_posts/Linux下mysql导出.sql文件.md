---
title: Linux下mysql导出.sql文件
date: 2021-09-07 13:24:02
tags: [Linux, Mysql, Program]
categories: Linux
cover: https://z3.ax1x.com/2021/10/08/59yDF1.jpg
copyright_author: B1ackSand
copyright_author_href: https://github.com/B1ackSand
copyright_url: https://blacksand.top/2021/09/07/Linux下mysql导出.sql文件
copyright_info: 此文章版权归B1ackSand所有，如有转载，请注明来自原作者
---

在终端直接使用：

`mysqldump -u root -p abc >abc.sql`

此处abc为目标数据库，abc.sql为存储文件名。

文件会导出到执行命令的目录下。

