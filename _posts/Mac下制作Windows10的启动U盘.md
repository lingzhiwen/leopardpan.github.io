# mac  制作windows启动U盘

## 1.查看你的U盘序号

> diskutil list

## 2.格式化U盘， 并且重命名你的U盘为"WINDOWS10"

> diskutil eraseDisk MS-DOS "WINDOWS10" MBR diskN   // N代表你刚刚查看的U盘序号

## 3.挂载系统镜像ISO文件，然后你就可以看到镜像名称了

> ls /Volumes

我的镜像名称是 /Volumes/CCCOMA_X64FRE_ZH-CN_DV9，如果你的名称不是这个，请在后续的命令中替换成你的。

## 4.复制到U盘

查看镜像文件大小：
> ls -lh /Volumes/CCCOMA_X64FRE_ZH-CN_DV9/sources/install.wim

**如果小于4 GB** ，您可以按照以下步骤复制所有文件，那么过程很简单（完成后只需将USB磁盘弹出到Finder中即可）
> rsync -avh --progress /Volumes/CCCOMA_X64FRE_ZH-CN_DV9/ /Volumes/WINDOWS10

**如果大于4 GB** ,（似乎在所有最近的Windows 10下载中都存在），则需要拆分文件。复制除 install.wim USB驱动器外的所有文件

> rsync -avh --progress --exclude=sources/install.wim /Volumes/CCCOMA_X64FRE_ZH-CN_DV9/ /Volumes/WINDOWS10

然后下载wimlib

> brew install wimlib

并使用它分割install.wim

>wimlib-imagex split /Volumes/CCCOMA_X64FRE_ZH-CN_DV9/sources/install.wim /Volumes/WINDOWS10/sources/install.swm 3800

这3800意味着该文件应分为3800 MB大小的块（以保持在4 GB以下的限制）。

只需在Finder中WINDOWS10单击弹出符号即可弹出该卷，然后卸下USB驱动器。现在可以将其用作可引导安装磁盘。
