---
layout: post
title: 移动Visual Studio的Package Cache文件夹
---

最近有个项目需要用到VS，只好忍痛安装了VS3012 community update 4.说实话 **VS是个好东西**，虽然我选择安装在D盘，但是安装完后还是让我的C盘减少了近11G的空间。前几天好不容易才腾出来的一下C盘又变红了。

下面找到一个方法可以节省一习C盘的空间。通过乱点发现，在 C:\ProgramData\Package Cache 这个文件夹有好几个G，并且看起来很可疑，就是安装VS当天生成的。

下面我们把这个文件夹的东西从系统盘挪出去，放到 F:\AppData\Package Cache 目录。 删掉原来的 Package Cache文件夹。

其实，这个文件夹和vs的修复与卸载这些功能有关系，直接挪走或者删除的话会导致vs不能正确卸载，下面我们生成一个链接文件，让vs以为文件还在这里。

1. 打开 Windows的命令行窗口，最好是权限提升的命令行窗口。
2. 执行 ```mklink /J "C:\ProgramData\Package Cache" "F:\AppData\Package Cache"```
3. Done.这样C盘终于又留出来一些空间了。

其实有不少程序的缓存文件夹可以这样处理来从系统盘腾空间，比如Chrome，但是由于我只是系统盘使用了SSD，这样处理可能享受不到SSD带来的速度优势了，这些可以自己去摸索与权衡了

[参考](http://superuser.com/questions/455853/can-i-delete-the-the-folder-c-programdata-package-cache)

更详细的参考:[Moving Package Cache](http://flightsimeindhoven.nl/?page_id=10371)