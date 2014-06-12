http://www.anheng.com.cn/loongson/install/readme.utf8.txt
https://wiki.debian.org/DebianYeeloong/HowTo/Install

欢迎使用社区版的龙芯操作系统, 本系统基于debian , 只进行了必要的调整, 以适应龙芯硬件

0.适用于龙梦公司的产品。龙芯2E盒子,6002,6003,6004,6005,6014,6015,8089_C,8089_D...
  以后升级建议直接使用系统内部的升级程序，跟用新版本的安装程序安装系统的效果一样。
  所有的文件从 http://www.anheng.com.cn/loongson/install 下载
  如果安装本系统后， 再恢复回原来的官方系统， 请先备份数据， 因为官方系统的安装程序， 
  不能识别出本系统的home所在的分区，即使提示不会破坏home， 也会把本系统的home清理掉

1.选择下载lzma后缀的打包文件，还有vmlinux文件,
  放到u盘的根目录，u盘不要大于2G,如果大于2G，可以分出一个1G的主分区使用，u盘使用的格式，2F盒子和笔记本用fat32格式,2E盒子使用ext3格式。
  如果需要升级pmon还可以选择下载相应的pmon文件。
2.将此u盘插入龙芯电脑。2E盒子需要插在4个usb口的上面2个usb插口位置上。 
    2E的usb插口比较深，最好用延长线来接u盘，否则会接触不上
　　2E的bios对大容量u盘支持不好，　最好用个小容量u盘载入vmlinux,然后压缩包放在另一个u盘上。 
3. 有3种方式可以开始安装,最简单的是第一种:
3.1 第一种载入方式:在龙芯笔记本，可以在开机时按tab，安装本系统。

3.2 第二种:bios菜单方式:新版的pmon有一个类似于bios的菜单， 可以按B键出来，
   移动光标到【Boot】列，
   【Set disk】这里回车选usb0
   【Set file type】这里回车选 fat或者ext2
   【Set karg】 设为console=tty
   【Set kernel path】设为/vmlinux ，然后【Boot Now】
  或者也可以在【exit】选择【exit to pmon】进入pmon命令行，用3b的步骤载入

3.3 第三种:pmon命令行方式，需要手动载入,适用于盒子的早期版本的pmon:
  首先开机时按del， 进入pmon命令模式
  2F盒子(fat):
  pmon> load /dev/fat/disk@usb0/vmlinux
  pmon> g
  2F盒子(ext2,3):
  pmon> load /dev/fs/ext2@usb0/vmlinux
  pmon> g
  2E盒子(ext2,3):
  pmon> load /dev/fs/ext2@usb0/vmlinux-2e
  pmon> g
  如果不能load, 还可以试试：
　pmon> load (usb0,0)/vmlinux
  pmon> g
  如果不行， 还可以把usb0 换成usb1试试
  还不行的话， 试试用一个小的u盘,比如128m，放上vmlinux ， 载入后， 换大的u盘, 然后执行g.

4.安装过程中,原来的所有数据会被清除，请事先备份。
5.程序会扫描所有插在系统上的u盘，sd卡，列出上面存放的压缩包，
    让您选择从哪个压缩包安装系统。

6.可以更换u盘， 用rescan命令重新扫描文件。
7.在清除您的数据前，需要输入2次Yes来确认。注意大小写。
8.压缩包的命名格式为：  loongson*201*.tar.*,比如:
  loongson2_debian6_20100119.tar.gz
  loongson2_gentoo_20100122.tar.bz2
  loongson2_debian6_20100615.tar.lzma

9.如果压缩包是gz或者lzma格式，大约10分钟可以完成安装。
9a.如果使用的pmon的版本比较低(盒子小于1.36a,笔记本小于1.49a)，载入initrd.img会特别慢，
   可以修改grub.cfg或者boot.ini,不载入initrd.img 或者升级pmon 
10.如果要测试android1.5，可以启动后连通网络，执行桌面上的install_android
11.如果需要安装游戏， 可以在启动后,联通网络后，执行桌面上的install_game
12.下载地址:
   下载vmlinux和一个大的压缩包.lzma，放到u盘:
   http://www.anheng.com.cn/loongson/install
   ftp://www.bjlx.org.cn/loongson/install
13.笔记本打开无线的热键是 Fn-F5, 打开无线网卡后， 右下角3个灯的右面一个会亮。
14.硬盘剩余空间，可以用cfdisk进行分区， 然后用mkfs.ext4格式化，然后mount使用。  
15.Fn-ESC 可以打开摄像头, 打开后,会出现/dev/video0的V4l的视频设备.   
   最后，备份好你的数据，让我们进入debian6的世界！
          http://www.bjlx.org.cn 20100315


最后更新:
20100430 修复龙芯盒子不能启动问题。
20100315 ffmpeg加速 yeeloong显卡加速
20100119 支持2E盒子
20100114 改用lzma压缩方式

另:
选lzma压缩格式的原因:
选用lzma压缩格式，在大幅度减小体积的同时，解压缩的时间没有明显延长：
gz->lzma    830M->550M  4分钟－>5分钟
lzma 解压缩命令: lzma -dc 20100122.tar.lzma |tar xv
或者 lzmadec -c 20100122.tar.lzma |tar xv

pmon的版本：
在pmon命令行输入set可以看到，
pmon升级命令：
2F盒子(u盘fat):
load -r -f bfc00000 /dev/fat/disk@usb0/pmon-LM60xx-1.3.6a.bin
2F盒子(u盘ext2,3):
load -r -f bfc00000 /dev/fs/ext2@usb0/pmon-LM60xx-1.3.6a.bin
2F笔记本:
load -r -f bfc00000 (usb0,0)/pmon-LM8089-1.4.9a.bin
2E盒子(u盘ext2,3):
load -r -f bfc00000 /dev/fs/ext2@usb0/pmon_2e.bin
２F笔记本的EC固件:
load -r -d /dev/fs/ext2@usb0/pq1d26.bin
升级后,要断电, 取下电池,重新装上, 刷新后,如果显示很暗，请按fn+向上的光标键,提高亮度,或者fn-f2 切换
EC和pmon版本, 可以在启动后, 通过 cat /proc/cmdline 查看.


名词解释:
pmon:龙芯的bootloader, 功能相当于pc机的bios+grub.  
 用于对龙芯进行初始化，显示logo, 并且从硬盘载入启动菜单， 或者直接载入内核，并启动。
2F盒子:龙梦公司的fulong minipc，型号有:6002,6003,6004
lzma:文件压缩格式
EC固件:龙梦笔记本里的一个监控芯片需要加载的程序, 直接在pmon命令行下烧写进去
