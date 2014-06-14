## 三 编译OpenWrt

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



### 2. 添加扩展目标平台(Add targets from extra feeds)

添加额外的扩展硬件平台：

    $ cp targets.conf.default targets.conf

然后编辑targets.conf文件，前面加注释符号＃表示该软件源不会被下载和安装。需要安装何种软件源就去掉相应的＃：


        src-git loongson2f git://github.com/openosom/loongson2f_openwrt10.03.1
        #src-link new-platform /usr/src/openwrt/target-feed


现在，更新下载这些硬件设备，并安装它们到target/linux/目录下：

    $ ./scripts/targets update -a
    $ ./scripts/targets install -a

  * 更多详情参看：[扩展硬件平台target feeds详细说明]()



### 3. 配置OpenWrt（Configure target and packages）

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


__比如要为TP-LINK TL-MR3420编译OpenWrt：__

  * 1 选择CPU型号

        Target System   --->   (X) Atheros AR71xx/AR7240/AR913x

<img src="/md/img/openwrt/compile/compile_001.png" alt="OpenWrt" align="middle" width="480" height="300"/>
<img src="/md/img/openwrt/compile/compile_002.png" alt="" align="middle" width="480" height="300"/>


  * 2 选择路由型号

        Target Profile  --->   (X) TP-LINK TL-MR3420 v1

<img src="/md/img/openwrt/compile/compile_003.png" alt="OpenWrt" align="middle" width="480" height="300"/>
<img src="/md/img/openwrt/compile/compile_004.png" alt="" align="middle" width="480" height="300"/>


  * 3 软件包就使用默认的，或者可以根据需要添加。
    - a. 添加luci

            LuCI ->Collections -> <*> luci

    - b. 添加luci的中文语言包

            LuCI ->Translations -> <*> luci-i18n-chinese

    - c. 添加DDNS

            LuCI ->Applications -> <*>luci-app-ddns

    - d. 添加USB挂载

            Base system -> <*>block-mount

    - e. 添加硬盘格式支持

            Kernel modules -> Filesystems -> <*> kmod-fs-ext4

    - f. 添加UTF8编码

            Kernel modules -> Native Language Support -> <*> kmod-nls-utf8

    - g. 添加USB扩展支持

            Kernel modules -> USB Support -> <*> kmod-usb-storage-extras.

    - h. 添加自动挂载工具

            Utilities -> Filesystem -> <*> badblocks

    - i. 添加复位键支持

            Utilities -> <*> restorefactory

    - j. 添加一键开关无线

            Utilities -> <*> wifitoggle



  * 如果需要单独配置OpenWrt的linux kernel，可以使用下面命令配置：

        $ make kernel_menuconfig

   这和普通的linux kernel的make menuconfig没有什么区别，可以根据需求自行配置。

  * 如无特殊情况，直接使用默认的内核配置即可！



### 4. 编译OpenWrt（Build OpenWrt image）

  * 完成menuconfig配置后，退出保存，开始编译OpenWrt：

        $ make


  * 编译过程可能会很久(一边下载源代码一边编译)，请确认你有足够的磁盘空间（应至少保证能够有大约4G以上的空间）。

<img src="/md/img/openwrt/compile/compile_005.png" alt="" align="middle" width="480" height="300"/>



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



### 5. 烧写固件（Flash）

  * 编译结束后，生成的固件存放在bin目录下，其中包含了kernel和rootfs的镜像文件，以及众多可以使用OPKG工具安装的.ipk安装包。
  * 接下了就是往flash中烧写固件，每个硬件设备不尽相同，不在详述。

  这是选择编译LUCI软件包，登录到管理界面：

<img src="/md/img/openwrt/compile/compile_luci.png" alt="" align="middle" width="480" height="300"/>



### 更多参考

  * [OpenWrt Buildroot - Usage](http://wiki.openwrt.org/doc/howto/build)
  * [OpenWrt Developer and User Documentation](http://downloads.openwrt.org/kamikaze/docs/openwrt.html )
  * [OpenWrt Docs on the Wiki](http://wiki.openwrt.org)
  * [OpenWrt Development Center](https://dev.openwrt.org)



