## 三 编译目标平台goldfish

__编译目标平台xxx系列__ 是[第三章 编译OpenWrt-DreamBox](/blog/#!/2014/06/09/OpenWrt开发文档-编译OpenWrt)的延续，将对各个不同的目标平台进行详细描述。

----------------------

  * 已经把goldfish放在OpenWrt-OpenOSOM中得到支持，主要针对嵌入式开发的。请参考[准备工作](/blog/#!/2014/06/09/OpenWrt开发文档-准备工作)和[获取OpenWrt-OpenOSOM源代码](/blog/#!/2014/06/09/OpenWrt开发文档-获取OpenWrt源代码)之后就可以开始编译工作了^_^


  * 目前goldfish使用的是ARMv5指令集，linux-2.6.30内核，其模拟器qemu-goldfish也被我收录在feeds/device下。


  * 哈哈，在没有嵌入式硬件开发平台的情况下，我们也能利用OpenWrt做嵌入式开发啦 : )


  * 本篇会详细描述：
   - a. qemu-goldfish模拟器
   - b. tslib触摸屏校正：该软件包在feeds/qpe中
   - c. qtopia图形界面：该软件包在feeds/qpe中
   - d. gcc编译器：该软件包刚刚被我修改完成，在feeds/packages_10.03.2中


  * 首先确认安装了如下feeds：

        src-svn packages svn://svn.openwrt.org.cn/dreambox/feeds/packages_10.03.2
        src-svn qpe svn://svn.openwrt.org.cn/dreambox/feeds/qpe
        src-svn device svn://svn.openwrt.org.cn/dreambox/feeds/device


  * 现在，更新下载这些包，并安装它们到package/feeds/目录：

        $ ./scripts/feeds update -a
        $ ./scripts/feeds install -a

  * 更多详情参看：[扩展软件包package feeds详细说明]()




### 配置OpenWrt（Configure target and packages）


现在开始配置OpenWrt，选择你需要目标平台，以及选择将要编译和添加进固件的软件包。运行命令：

        $ make menuconfig


  * 1 选择CPU型号

        Target System   --->    (X) Goldfish (Android Emulator)

<img src="/md/img/openwrt/goldfish/goldfish_menu_target_01.png" alt="loongson2f" align="middle" width="480" height="300"/>
<img src="/md/img/openwrt/goldfish/goldfish_menu_target_02.png.png" alt="loongson2f" align="middle" width="480" height="300"/>


  * 2 Target Profile你可以选择不同的默认软件配置：

        Target Profile  --->   (X) (qtopia)Goldfish (Android Emulator)

   - a. (minimal)Goldfish (Android Emulator) : 这个配置仅仅包含最基本的packages，可能只有shell和一些命令行程序。
   - b. (qtopia)Goldfish (Android Emulator) : 包含qtopia作为桌面环境，以及tslib触摸屏校验等。

<img src="/md/img/openwrt/goldfish/goldfish_menu_profile_01.png.png" alt="loongson2f" align="middle" width="480" height="300"/>
<img src="/md/img/openwrt/goldfish/goldfish_menu_profile_02.png.png" alt="loongson2f" align="middle" width="480" height="300"/>


  * 3 默认已经添加了qemu_goldfish模拟器：

        Device -> <*> qemu_goldfish

<img src="/md/img/openwrt/goldfish/goldfish_qemu_01.png" alt="loongson2f" align="middle" width="480" height="300"/>
<img src="/md/img/openwrt/goldfish/goldfish_qemu_02.png" alt="loongson2f" align="middle" width="480" height="300"/>



  * 4 选择build-essential，它会自动把binutils，gcc，make等都选择上，这样就能直接编译出goldfish所使用的Native Toolchain本地编译工具链了！

        Development  ---> <*> build-essential

<img src="/md/img/openwrt/goldfish/goldfish_menu_gcc_01.png.png" alt="loongson2f" align="middle" width="480" height="300"/>
<img src="/md/img/openwrt/goldfish/goldfish_menu_gcc_02.png.png" alt="loongson2f" align="middle" width="480" height="300"/>



### 2. 编译OpenWrt（Build OpenWrt image）


  * 完成menuconfig配置后，退出保存，开始编译OpenWrt：

        $ make


  * 编译过程可能会很久(一边下载源代码一边编译)，请确认你有足够的磁盘空间（应至少保证能够有大约4G以上的空间）。


  * 另外，你可以使用:

        $ make -j 3 V=99

这个命令会使用双核CPU进行编译，并且输出编译过程中的信息，这样你可以比较清楚的看到编译情况或出错情况：

  * -j3：具有多核CPU处理器的PC进行编译，可使用-j参数加速编译，用法为-j <cpu处理器的数目 + 1>
  * V=99：表示输出详细的debug信息
  * make world：表示编译所有



### 4. goldfish运行OpenWrt

  * 编译结束后，生成的固件存放在bin/goldfish目录下，其中包含了kernel和rootfs的镜像文件等，以及众多可以使用OPKG工具安装的.ipk安装包。
   - a. qemu-goldfish： 该目录下包含goldfish的模拟器已经所使用的皮肤
   - b. run-emulator.sh： 运行qemu-goldfish的脚本文件
   - c. packages： 其中包含了众多可以使用OPKG工具安装的.ipk安装包
   - d. openwrt-goldfish-kernel.bin等： 为qemu-goldfish所使用的kernel，system，data，ramdisk，rootfs等


  * 进入bin/goldfish/，运行模拟器

        $ cd bin/goldfish/
        $ ./run-emulator.sh

<img src="/md/img/openwrt/goldfish/goldfish_run_02.png" alt="loongson2f" align="middle" width="480" height="300"/>
<img src="/md/img/openwrt/goldfish/goldfish_run_03.png" alt="loongson2f" align="middle" width="480" height="300"/>


  * 测试运行goldfish下的本地编译器gcc

        $ cd /usr/sr/hello/
        $ gcc hello.c -o hello
        $ ./hello
        
                Hello OpenWrt

<img src="/md/img/openwrt/goldfish/goldfish_run_06.png" alt="loongson2f" align="middle" width="480" height="300"/>

  * 运行tslib触摸屏校正

        $ . /etc/tslib-env.sh
        $ ts_calibrate

<img src="/md/img/openwrt/goldfish/goldfish_run_04.png" alt="loongson2f" align="middle" width="480" height="300"/>
<img src="/md/img/openwrt/goldfish/goldfish_run_05.png" alt="loongson2f" align="middle" width="480" height="300"/>


  * 运行qtopia图形界面

        $ . /etc/tslib-env.sh
        $ . /etc/qtopia-env.sh
        $ qpe

<img src="/md/img/openwrt/goldfish/goldfish_qpe_01.png" alt="loongson2f" align="middle" width="480" height="300"/>
<img src="/md/img/openwrt/goldfish/goldfish_qpe_02.png" alt="loongson2f" align="middle" width="480" height="300"/>
<img src="/md/img/openwrt/goldfish/goldfish_qpe_03.png" alt="loongson2f" align="middle" width="480" height="300"/>
<img src="/md/img/openwrt/goldfish/goldfish_qpe_04.png" alt="loongson2f" align="middle" width="480" height="300"/>





---------------------------------------

  * 转载请保留信息完整性，来源于 [http://www.openosom.org/blog/](http://www.openosom.org/blog/)




