## 三 编译目标平台loongson2f

在完成[准备工作](/blog/#!/2014/06/09/OpenWrt开发文档-准备工作)和[获取OpenWrt-OpenOSOM源代码](/blog/#!/2014/06/09/OpenWrt开发文档-获取OpenWrt源代码)之后就可以开始编译工作了^_^

  * 编译OpenWrt一般需要以下五个步骤：
    1. 准备工作：搭建编译环境
    2. 通过svn、git获取OpenWrt源代码
    3. 更新下载和安装扩展软件包，即package feeds
    4. 用menuconfig来配置目标平台和软件包
    5. 最后make开始编译固件


  * 假设你的工作目录为openosom，进入openosom目录：

        $ cd openosom

  * OpenWrt-OpenOSOM会不停地更新，为了保证源代码的最新可以使用svn up或者git pull更新：

        $ svn update    //如果是SVN代码仓
        $ git pull      //如果是GIT代码仓



### 1. 添加扩展软件包(Add packages from extra feeds)

虽然基本系统可以build出Image，但生成的Image缺少很多应用程序，需要添加额外的扩展软件包：

    $ cp feeds.conf.default feeds.conf

然后编辑feeds.conf文件，前面加注释符号＃表示该软件源不会被下载和安装。需要安装何种软件源就去掉相应的＃：


        src-svn packages svn://svn.openwrt.org.cn/dreambox/feeds/packages_10.03.2
        #src-svn luci http://svn.luci.subsignal.org/luci/tags/0.10.0/contrib/package
        #src-link custom /usr/src/openwrt/custom-feed


请注意，这个设置并不表示这些应用程序包会被编译，而只是在后面使用menuconfig命令时会获取这些包的描述信息（OpenWrt-Makefiles）。

现在，更新下载这些包，并安装它们到package/feeds/目录：

    $ ./scripts/feeds update -a
    $ ./scripts/feeds install -a

  * 更多详情参看：[扩展软件包package feeds详细说明]()




### 2. 配置OpenWrt（Configure target and packages）

下一步是检查编译环境，若可进行编译则生成默认配置：

        $ make defconfig

若defconfig回显提示缺少软件包或编译库等依赖，则按提示安装所缺软件包或库等即可，不难的，细心点就行。

现在开始配置OpenWrt，选择你需要目标平台，以及选择将要编译和添加进固件的软件包。运行命令：

        $ make menuconfig

这个配置界面跟Linux内核的配置界面基本差不多，几乎每一项都有3个选择Y/M/N：

  * Y：选择Y，该软件将被编译，并且加入到你的目标固件里；
  * M：选择M，该软件包将会被编译，但不会被放入固件里。在需要它的时候，可以用OPKG软件包管理器进行安装；
  * N：选择N，该软件包将不会被编译，也不会被安装进固件。


  * 方向键是移动光标
  * 回车键是确认
  * 空格键是选择，可以代替Y/M/N键的使用



  * 1 选择为龙芯CPU型号

        Target System   --->    (X) LoongSon 2F

<img src="/md/img/openwrt/loongson2f/openwrt-loongson2f_001.png" alt="loongson2f" align="middle" width="480" height="300"/>


  * 2 根据需求选择已经默认安装不同软件包的选项，如mini、luci（wifi未完成）：

        Target Profile  --->   (X) YeeLoong2F(8089) LUCI
                               ( ) YeeLoong2F(8089) mini
                               ( ) YeeLoong2F(8089) wifi

<img src="/md/img/openwrt/loongson2f/openwrt-loongson2f_002.png" alt="loongson2f" align="middle" width="480" height="300"/>


  * 3 软件包就使用默认的，或者可以根据需要添加。例如在选mini的情况下，自己手动选上luci的软件包。
    - a. 添加luci

            LuCI ->Collections -> <*> luci

    - b. 添加luci的中文语言包

            LuCI ->Translations -> <*> luci-i18n-chinese

    - c. 添加DDNS

            LuCI ->Applications -> <*>luci-app-ddns

<img src="/md/img/openwrt/loongson2f/openwrt-loongson2f_003.png" alt="loongson2f" align="middle" width="480" height="300"/>



  * 如果需要单独配置OpenWrt的linux kernel，可以使用下面命令配置：

        $ make kernel_menuconfig

   这和普通的linux kernel的make menuconfig没有什么区别，可以根据需求自行配置。

  * 如无特殊情况，直接使用默认的内核配置即可！



### 3. 编译OpenWrt（Build OpenWrt image）

  * 完成menuconfig配置后，退出保存，开始编译OpenWrt：

        $ make


  * 编译过程可能会很久(一边下载源代码一边编译)，请确认你有足够的磁盘空间（应至少保证能够有大约4G以上的空间）。


  * 另外，你可以使用:

        $ make -j 3 V=99

这个命令会使用双核CPU进行编译，并且输出编译过程中的信息，这样你可以比较清楚的看到编译情况或出错情况：

  * -j3：具有多核CPU处理器的PC进行编译，可使用-j参数加速编译，用法为-j <cpu处理器的数目 + 1>
  * V=99：表示输出详细的debug信息
  * make world：表示编译所有

如只想清除/编译某个模块，您可以做如下类似操作：

        $ make package/qos/clean
        $ make package/qos/compile
        $ make package/qos/install



### 4. 安装OpenWrt到龙芯2F本

  * 编译结束后，生成的固件存放在bin目录下，其中包含了kernel和rootfs的镜像文件，以及众多可以使用OPKG工具安装的.ipk安装包。
  * 接下了就是在龙芯2F电脑上安装OpenWrt了。


这里使用龙芯2官方提供的U盘安装Debian的vmlinux，所以为了使得能够安装OpenWrt系统，需要如下操作：

  * 首先把bin/loongson2f/目录下的vmlinux-usbinstall和loongson2f-openwrt-rootfs-2014.tar.gz两个文件复制到U盘根目录。（U盘可为fat、ext2、ext3文件系统）

  * U盘插入龙芯电脑，开机时按del， 进入PMON命令模式，输入如下指令。

        pmon> load (usb0,0)/vmlinux-usbinstall
            
                Loading file:(usb0,0)/vmlinux-usbinstall\(elf)
                (elf)
                0x........
                Entry address is ...
            
        pmon> g


加载U盘上的内核（命令load (usb0,0)/vmlinux-usbinstall）并运行（命令g）。


NOTE：

  * 如果不行， 还可以把usb0 换成usb1试试；
  * 还不行的话， 试试用一个小的u盘,比如128m，放上vmlinux ， 载入后， 换大的u盘, 然后执行g。
  * 或者尝试如下命令，对于U盘是fat文件系统：

        pmon> load /dev/fat/disk@usb0/vmlinux

  * 对于U盘是ext2,3文件系统：

        pmon> load /dev/fs/ext2@usb0/vmlinux



  * 成功运行内核后，会进入如下界面：

        =======================================================
        /dev/sda1 loongson2f-openwrt-rootfs-2014.tar.gz xx M
        =======================================================
        
        /dev/hda All data will delete! continue? [yes/No/rescan/shell]
        yes  ## 输入yes，开始安装 ##
        
        /dev/hda All DATE WILL DELETE!!! continue? [yes/No]
        yes  ## 输入yes，确认安装 ##
        
        fstype: 1-ext4 2-btrfs
        Please select[1]: 1  ## 输入1或者2，选择相应的文件系统 ##


下面开始安装，安装好后会自动重启，在进入一次安装程序，设置OpenWrt的boot.cfg，从而PMON可以引导、启动OpenWrt。


  * 进入PMON命令行：

        pmon> load (usb0,0)/vmlinux-usbinstall
        pmon> g


  * 把含有boot.cfg的/dev/hda1  mount到mnt目录，设置boot.cfg:

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


okay，安装完毕！


  * 龙芯2F本上运行OpenWrt：

<img src="/md/img/openwrt/loongson2f/loongson2f-00-runopenwrt.jpg" alt="loongson2f" align="middle" width="300" height="400"/>


  * SSH登陆龙芯本的OpenWrt：

<img src="/md/img/openwrt/loongson2f/loongson2f-01-sshopenwrt.png" alt="loongson2f" align="middle" width="480" height="300"/>


  * 这是选择编译LUCI软件包，WEB登录到管理界面：

<img src="/md/img/openwrt/loongson2f/loongson2f-02-webluci01.png" alt="loongson2f" align="middle" width="480" height="300"/>


  * WEB登陆龙芯本的LUCI-状态总览界面：

<img src="/md/img/openwrt/loongson2f/loongson2f-02-webluci02.png" alt="loongson2f" align="middle" width="480" height="300"/>


  * WEB登陆龙芯本的LUCI-负载界面：

<img src="/md/img/openwrt/loongson2f/loongson2f-02-webluci03.png" alt="loongson2f" align="middle" width="480" height="300"/>









