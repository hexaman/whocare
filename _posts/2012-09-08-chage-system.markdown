---
layout: post
title: 更换系统
description: 这是 hexaman 在 GitHub 上面的第一篇文章，您好，世界。
keywords: hexaman,windows to linux,arch,lxde
---

电脑上原来的windowsXP老是用一会就死机，于是想重装。之前想先试下linux。于是发现，完全可以抛弃windows了。

现在是ArchLinux+LXDE，速度真的飞快。10年前刚认识linux时，发现X-Window系统是各种崩溃，现在真的很稳定了。用了几天，比微软的系统稳定多了。看了下，2G内存才用了几百兆。完全可以同时开20个视频窗口看爱情动作片嘛！！

下面是安装过程。(具体看<a href="https://wiki.archlinux.org/index.php/Beginners%27_Guide_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)">这里</a>)

##一，创建启动盘。

在windows下载archlinux-2012.08.04-dual.iso文件。  
$ sha1sum --check archlinux-2012.08.04-dual.txt   
把archlinux-2012.08.04-dual.iso复制到U盘(H:)新建的/Boot目录，从最新版syslinux-4.05.zip中复制./win32/syslinux.exe 到U盘根目录。./memdisk/memdisk 到 "/Boot" 目录。在"/Boot"目录下再创建一个 syslinux.cfg 文件，内容如下：  
    DEFAULT arch_iso  
    LABEL arch_iso  
            MENU LABEL Arch Setup  
            LINUX memdisk  
            INITRD /Boot/archlinux-2012.08.04-dual.iso  
            APPEND iso  

然后，在U盘根目录中创建一个bootusb.bat文件并运行，文件内容如下：  
    @echo off   
    syslinux.exe -m -a -d /Boot H:   

##二，启动USB 

把USB插入电脑启动，按F12,选Harddisk,选中U盘启动。

##三，安装系统

准备分区   
\# fdisk /dev/sda/  
\# mkfs.ext2 /dev/sda1  
\# mkfs.ext4 /dev/sda5  
\# mkfs.ext4 /dev/sda9  
\# mount /dev/sda5 /mnt  
\# mkdir /mnt/boot  
\# mount /dev/sda1 /mnt/boot  
\# mkdir /mnt/home  
\# mount /dev/sda9 /mnt/home  

准备源  
\# pppoe-setup  
\# pppoe-start  
\# nano /etc/resolv.conf 加入以下两行。  
nameserver 8.8.8.8  
nameserver 4.4.4.4  
\# ping -c 3 baidu.com   \#测试网络  
\# nano /etc/pacman.d/mirrorlist，将中国境内的镜像放到前面

安装基本系统  
\# pacstrap /mnt base base-devel  

生成 fstab  
\# genfstab -p /mnt >> /mnt/etc/fstab  

Chroot 到新系统  
\# arch-chroot /mnt  

配置系统  
\# nano /etc/hostname  
arch  
\# nano /etc/hosts  
    127.0.0.1   localhost.localdomain   localhost arch  
    ::1         localhost.localdomain   localhost arch  
\# nano /etc/vconsole.conf  
    KEYMAP=us  
    FONT=  
    FONT_MAP=  
\# /etc/timezone  
    Asia/Hong_Kong  
\# ln -s /usr/share/zoneinfo/Asia/Hong_kong /etc/localtime   
默认情况下 /etc/locale.gen 是一个仅包含注释文档的空文件。编辑后,这个文件将不会更改。每次glibc更新之后就会运行 locale-gen 一次， 重新生成 /etc/locale.gen 指定的本地化文件。  
选定你需要的本地化类型(移除前面的\＃即可), 比如中文系统可以使用:  
\# nano /etc/locale.gen  
zh_CN.gb18030  
zh_CN.gb2312  
zh_CN.gbk  
zh_CN.utf8  
zh_HK  
zh_HK.big5hkscs  
zh_HK.utf8  
zh_TW  
zh_TW.big5  
zh_TW.utf8  
\# locale-gen  
\# hwclock --systohc --utc  
一般情况下 udev 会自动加载需要的模块，大部分用户都不需要手动修改。这里只需要加入真正需要的模块。  
/etc/modules-load.d/中保存内核启动时加入模块的配置文件。每个配置文件已在/etc/modules-load.d/以\<program\>.conf的格式命名。配置文件中包含需要装入的内核列表，每个一行。空行和以 \# 或 ; 开头的行直接被忽略。  
在 /etc/rc.conf 的 DAEMONS 行加入的程序将在开机时自动启动。  
\# cat /etc/rc.conf |grep DAEMONS  
    DAEMONS=(syslog-ng !network @crond @alsa dbus hal @adsl @dropboxd)  
\# mkinitcpio -p linux  

安装配置启动加载器  
\# pacman -S syslinux  
编辑 /boot/syslinux/syslinux.cfg，将 / 指向正确的根分区  
\# cat /boot/syslinux/syslinux.cfg |grep APPEND  
	APPEND root=/dev/sda5 ro  
\# syslinux-install_update -iam  
最后，用 passwd 设置一个root密码：  
\# passwd  
卸载分区并重启系统  
\# exit  
\# umount /mnt/{boot,home,}  
\# reboot  

##四，安装配置X桌面

新的 Arch Linux 基本系统已经是可以工作的 GNU/Linux 环境了。用 root 登录，用 root 用户配置 pacman 并更新系统。  

在 Arch x86_64 上运行 32 位应用程序，在 /etc/pacman.conf 中加入如下内容以启用 [multilib] 源：  
[multilib]  
Include = /etc/pacman.d/mirrorlist  

\# pacman -Syy  
\# pacman-key --init  
\# pacman-key --populate archlinux  
\# pacman -Syu  

\# adduser      \#uselinux  
加入以下用户组audio,lp,optical,storage,video,wheel,games,power,scanner  

\# pacman -S sudo  
\# visudo 将wheel组前的\#号取消。  

安装 X  
\# pacman -S xorg-server xorg-xinit xorg-utils xorg-server-utils mesa dbus xorg-twm xorg-xclock xterm  
安装显卡驱动  
\# pacman -S xf86-video-intel  
安装字体  
\# pacman -S ttf-dejavu  
\# pacman -S wqy-zenhei  
安装 LXDE  
\# pacman -S lxde  

安装完成后, 复制/etc/xdg/openbox里的3个文件到 ~/.config/openbox :  
\# mkdir -p ~/.config/openbox  
\# cp /etc/xdg/openbox/menu.xml /etc/xdg/openbox/rc.xml /etc/xdg/openbox/autostart ~/.config/openbox  

\# pacman -S leafpad obconf epdfview gamin  

编辑/home/uselinux/.bash_profile  
    # nano /home/uselinux/.bash_profile  
    # cat /home/uselinux/.bash_profile  
     #  
    # ~/.bash_profile  
     #  

    [[ -f ~/.bashrc ]] && . ~/.bashrc  

    if [[ -z $DISPLAY ]] && [[ $(tty) = /dev/tty1 ]]; then  
      #exec startx  
      # Could use xinit instead of startx  
      xinit -- /usr/bin/X -nolisten tcp vt7  
    fi  

    if [[ -z $DISPLAY ]] && ! [[ -e /tmp/.X11-unix/X0 ]] && (( EUID )); then  
      while true; do  
        read -p 'Do you want to start X? (y/n): '  
        case $REPLY in  
          [Yy]) xinit -- /usr/bin/X -nolisten tcp vt7 ;;  
          [Nn]) break ;;  
          *) printf '%s\n' 'Please answer y or n.' ;;  
        esac  
      done  

编辑/home/uselinux/.xinitrc  
    # nano /home/uselinux/.xinitrc  
    # cat /home/uselinux/.xinitrc  
    #  
    #!/bin/sh  
    #  
    # ~/.xinitrc  
    #  
    # Executed by startx (run your window manager from here)  

    #Locale settings  
    export LC_ALL=zh_CN.utf-8  
    export LANGUAGE=zh_CN.utf-8  
    export LANG=zh_CN.utf-8  

    #Start the dbus user session  
    if [ -d /etc/X11/xinit/xinitrc.d ]; then  
      for f in /etc/X11/xinit/xinitrc.d/*; do  
        [ -x "$f" ] && . "$f"  
      done  
      unset f  
    fi  

    #using LXDE for default desktop  
    exec ck-launch-session dbus-launch startlxde  

至此，全部完成。重启电脑。以uselinux用户登录。
