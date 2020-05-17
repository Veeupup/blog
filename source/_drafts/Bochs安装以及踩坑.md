---
title: Bochs安装以及踩坑
date: 2020-02-11 14:48:30
---

因为想要写一个简单的操作系统，所以需要安装虚拟机来模拟出硬件，VMware不适合这个场景，因为会使用硬件级别的虚拟化，而bochs这个开源虚拟机，是用软件虚拟了所有的硬件，所以调试可以做到非常细的粒度，比如每次cpu执行命令，我们都可以暂停，看寄存器状态，看内存状态，这对于操作系统开发调试的帮助太大太大了。所以我们使用bochs这个虚拟机来。

<!--more-->

## mac安装bochs

我的当前环境是Mac 版本是10.15.3，记录下安装的过程和踩的坑。

步骤：

* 安装 sdl 库，后续编译会用到

```shell
brew install sdl
```

* 下载源码并且解压

下载地址：https://sourceforge.net/projects/bochs/files/latest/download

```shell
# 下载解压
tar -xvf bochs-2.6.tar.gz
# configure 所需要的参数

./configure --enable-ne2000 \
            --enable-all-optimizations \
            --enable-cpu-level=6 \
            --enable-x86-64 \
            --enable-vmx=2 \
            --enable-pci \
            --enable-usb \
            --enable-usb-ohci \
            --enable-e1000 \
            --enable-debugger \
            --enable-disasm \
            --disable-debugger-gui \
            --with-sdl \
            --prefix=$HOME/opt/bochs

```

这里的 prefix 参数指定了安装的位置，修改成自己想要的地址。

这里出现了第一个问题：

### 问题1

报这个错误

```shell
cdrom_osx.cc:194:18: error: assigning to 'char ' from incompatible type 'const ch
```

于是在网上查了一下，这个报错有个补丁[https://raw.githubusercontent.com/Homebrew/formula-patches/e9b520dd4c/bochs/xcode9.patch]，我们找到这个文件并且修改源码，这个文件在 `bochs-2.6/iodev/hdimage/cdrom_osx.cc`，我们打开修改第 193 行

```c
if ((devname = strrchr(devpath, '/')) != NULL) {
改为：
if ((devname = (char *) strrchr(devpath, '/')) != NULL) {
```

问题解决，我们接着编译。

```shell
make && make install
```

### 问题二

这个时候报这个错

```shell
config.cc:3261:55: error: ordered comparison between pointer and zero ('char *' and 'int')
    if (SIM->get_param_string("model", base)->getptr()>0) {
        ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~^~
1 error generated.
```

我们打开config.cc，找到3621行，修改为

```c
修改config.cc的3621行
if (SIM->get_param_string("model", base)->getptr()>0) {
为
if (SIM->get_param_string("model", base)->getptr()>(char *)0) {
```

即可。

然后我们再次编译

```shell
make && make install
```

此时即可正常编译成功。

我们可以将 bochs 加入环境变量以便使用：

```shell
export BXSHARE="$HOME/workplace/os/bochs/share/bochs"
export PATH="$PATH:$HOME/workplace/os/bochs/bin"
```

## Bochs使用

bochs的安装目录下的`bochs/share/doc/bochs/bochsrc-sample.txt`是配置文件的模板。我们自己编写一个简单的配置文件来运行：

```
# 设置虚拟机内存为32MB
megs: 32

# 设置BIOS镜像
romimage: file=$BXSHARE/BIOS-bochs-latest 

# 设置VGA BIOS镜像
vgaromimage: file=$BXSHARE/VGABIOS-lgpl-latest

# 设置从硬盘启动
boot: disk

# 设置日志文件
log: bochsout.txt

# 关闭鼠标
mouse: enabled=0

# 打开键盘
keyboard: type=mf, serial_delay=250

# 设置硬盘
ata0: enabled=1, ioaddr1=0x1f0, ioaddr2=0x3f0, irq=14

# 添加gdb远程调试支持
gdbstub: enabled=1, port=1234, text_base=0, data_base=0, bss_base=0
```

保存为 bochsrc 。在该目录运行命令执行：

```shell
➜  my_src git:(master) ✗ bochs
========================================================================
                        Bochs x86 Emulator 2.6
            Built from SVN snapshot on September 2nd, 2012
                  Compiled on Feb 11 2020 at 14:52:19
========================================================================
00000000000i[     ] reading configuration from bochsrc
00000000000p[     ] >>PANIC<< bochsrc:26: Bochs is not compiled with gdbstub support
========================================================================
Bochs is exiting with the following message:
[     ] bochsrc:26: Bochs is not compiled with gdbstub support
========================================================================
00000000000i[CPU0 ] CPU is in real mode (active)
00000000000i[CPU0 ] CS.mode = 16 bit
00000000000i[CPU0 ] SS.mode = 16 bit
00000000000i[CPU0 ] EFER   = 0x00000000
00000000000i[CPU0 ] | EAX=00000000  EBX=00000000  ECX=00000000  EDX=00000000
00000000000i[CPU0 ] | ESP=00000000  EBP=00000000  ESI=00000000  EDI=00000000
00000000000i[CPU0 ] | IOPL=0 id vip vif ac vm rf nt of df if tf sf ZF af PF cf
00000000000i[CPU0 ] | SEG sltr(index|ti|rpl)     base    limit G D
00000000000i[CPU0 ] |  CS:0000( 0000| 0|  0) 00000000 00000000 0 0
00000000000i[CPU0 ] |  DS:0000( 0000| 0|  0) 00000000 00000000 0 0
00000000000i[CPU0 ] |  SS:0000( 0000| 0|  0) 00000000 00000000 0 0
00000000000i[CPU0 ] |  ES:0000( 0000| 0|  0) 00000000 00000000 0 0
00000000000i[CPU0 ] |  FS:0000( 0000| 0|  0) 00000000 00000000 0 0
00000000000i[CPU0 ] |  GS:0000( 0000| 0|  0) 00000000 00000000 0 0
00000000000i[CPU0 ] | EIP=00000000 (00000000)
00000000000i[CPU0 ] | CR0=0x00000000 CR2=0x00000000
00000000000i[CPU0 ] | CR3=0x00000000 CR4=0x00000000
bx_dbg_read_linear: physical memory read error (phy=0x0000000000000000, lin=0x0000000000000000)
00000000000i[CTRL ] quit_sim called with exit code 1
```

此时我们便成功运行了一个虚拟机。

### 模拟硬盘

bochs提供了一个创建虚拟硬盘的工具 bximage，提供了交互的方式创建虚拟硬盘，我们来做一下：

```shell
➜  my_src git:(master) ✗ bximage
========================================================================
                                bximage
                  Disk Image Creation Tool for Bochs
          $Id: bximage.c 11315 2012-08-05 18:13:38Z vruppert $
========================================================================

Do you want to create a floppy disk image or a hard disk image?
Please type hd or fd. [hd] // 回车，新建硬盘

What kind of image should I create?
Please type flat, sparse or growing. [flat]	// 回车，flat硬盘

Enter the hard disk size in megabytes, between 1 and 8257535
[10] 10	// 10MB大小硬盘

I will create a 'flat' hard disk image with
  cyl=20
  heads=16
  sectors per track=63
  total sectors=20160
  total size=9.84 megabytes

What should I name the image?
[c.img] hd10m.img	// 名称

Writing: [] Done.

I wrote 10321920 bytes to hd10m.img.

The following line should appear in your bochsrc:
  ata0-master: type=disk, path="hd10m.img", mode=flat, cylinders=20, heads=16, spt=63
➜  my_src git:(master) ✗ ls
bochsrc   hd10m.img
```

好了，环境配置到此结束，我们之后将学习如何编写启动信息。