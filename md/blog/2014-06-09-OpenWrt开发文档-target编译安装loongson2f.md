

这里使用龙芯2官方提供的U盘安装Debian的vmlinux，所以为了使得能够安装OpenWrt系统，需要如下操作：

首先开机时按del， 进入pmon命令模式，输入如下指令。
pmon> load (usb0,0)/vmlinux-usbinstall

Loading file:(usb0,0)/vmlinux-usbinstall\(elf)
(elf)
0x........
Entry address is ...

pmon> g
加载U盘上的内核并运行。

NOTE：
  如果不行， 还可以把usb0 换成usb1试试；
  还不行的话， 试试用一个小的u盘,比如128m，放上vmlinux ， 载入后， 换大的u盘, 然后执行g。
或者尝试如下命令，对于U盘是fat文件系统：
  pmon> load /dev/fat/disk@usb0/vmlinux
对于U盘是ext2,3文件系统：
  pmon> load /dev/fs/ext2@usb0/vmlinux




=======================================================
/dev/sda1 loongson2f-openwrt-rootfs-2014.tar.gz xx M
=======================================================
/dev/hda All data will delete! continue? [yes/No/rescan/shell]
yes

/dev/hda All DATE WILL DELETE!!! continue? [yes/No]
yes

fstype: 1-ext4 2-btrfs
Please select[1]:
1
下面开始安装，安装好后会自动重启。




pmon> load (usb0,0)/vmlinux-usbinstall
pmon> g

=======================================================
/dev/sda1 loongson2f-openwrt-rootfs-2014.tar.gz xx M
=======================================================
/dev/hda All data will delete! continue? [yes/No/rescan/shell]
shell

mkdir -pv /mnt
mount /dev/hda1 /mnt/
cd /mnt/
cp boot.cfg.openwrt boot.cfg
reboot


