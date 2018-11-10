---
layout: post
title: 使用u盘安装ubuntu16.04的那些事儿
categories:
- Technology
tags:
- 系统
---


# 使用upan系统制作工具
* 不要使用ultraISO工具
* 使用 [easyBCD](http://neosmart.net/EasyBCD/) 或者 [Universal-USB-Installer](https://www.pendrivelinux.com/universal-usb-installer-easy-as-1-2-3/) 工具

# 系统安装完毕后

## 1、更系统源

### 1.1、 如何更改源

可以在软件更新中选择源,
使用如下命令更改(修改前先备份):

    sudo vim /etc/apt/source.list

### 1.2、 阿里云

```sh
deb http://mirrors.aliyun.com/ubuntu/ quantal main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ quantal-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ quantal-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ quantal-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ quantal-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ quantal main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ quantal-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ quantal-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ quantal-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ quantal-backports main restricted universe multiverse
```

### 1.3、 东北大学

```sh
deb-src http://mirror.neu.edu.cn/ubuntu/ xenial main restricted #Added by software-properties
deb http://mirror.neu.edu.cn/ubuntu/ xenial main restricted
deb-src http://mirror.neu.edu.cn/ubuntu/ xenial restricted multiverse universe #Added by software-properties
deb http://mirror.neu.edu.cn/ubuntu/ xenial-updates main restricted
deb-src http://mirror.neu.edu.cn/ubuntu/ xenial-updates main restricted multiverse universe #Added by software-properties
deb http://mirror.neu.edu.cn/ubuntu/ xenial universe
deb http://mirror.neu.edu.cn/ubuntu/ xenial-updates universe
deb http://mirror.neu.edu.cn/ubuntu/ xenial multiverse
deb http://mirror.neu.edu.cn/ubuntu/ xenial-updates multiverse
deb http://mirror.neu.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse
deb-src http://mirror.neu.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse #Added by software-properties
deb http://archive.canonical.com/ubuntu xenial partner
deb-src http://archive.canonical.com/ubuntu xenial partner
deb http://mirror.neu.edu.cn/ubuntu/ xenial-security main restricted
deb-src http://mirror.neu.edu.cn/ubuntu/ xenial-security main restricted multiverse universe #Added by software-properties
deb http://mirror.neu.edu.cn/ubuntu/ xenial-security universe
deb http://mirror.neu.edu.cn/ubuntu/ xenial-security multiverse
```

### 1.4、 清华大学

```sh
# deb cdrom:[Ubuntu 16.04 LTS _Xenial Xerus_ - Release amd64 (20160420.1)]/ xenial main restricted
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial main restricted
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates main restricted
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial universe
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates universe
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial multiverse
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates multiverse
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security main restricted
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security universe
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security multiverse
```


## 2、设置系统

### 2.1、设置unity的位置
参考[将 Ubuntu 16.04 LTS 的 Unity 启动器移动到桌面底部](http://www.linuxidc.com/Linux/2016-03/129359.htm)

从左侧移动感到底部:

    gsettings set com.canonical.Unity.Launcher launcher-position Bottom

恢复到左侧

    gsettings set com.canonical.Unity.Launcher launcher-position Left

### 2.2、点击unity启动器图标最小化

    gsettings set org.compiz.unityshell:/org/compiz/profiles/unity/plugins/unityshell/ launcher-minimize-window true

### 2.3、配置Dash

    “安全和隐私” 》 “搜索”， 打开在线搜索的功能
    “安全和隐私” 》 “诊断”，关闭向 Canonical 发送错误报告

### 2.4、配置快捷键

[参考文章](https://www.cnblogs.com/sink_cup/archive/2013/04/26/1987901.html)

#### 2.4.1、快速启动终端

```sh
设置快捷键
系统设置——》键盘——》快捷键。
启动器——》启动终端，设成Win + R。
```

#### 2.4.2、设置显示桌面快捷键
需要安装`compizconfig-settings-manager`软件

```sh
sudo apt-get install compizconfig-settings-manager
ccsm

ccsm——》桌面——》Ubuntu Unity Plugin。

General——》“Show Desktop”——》设置成“win + d”。
```

### 2.5、Ubuntu 切换中文目录或英文目录切换

* 切换为英文:

```sh
export LANG=en_US
xdg-user-dirs-gtk-update
echo 选 update
```

* 切换为中文:

```sh
export LANG=zh_CN.UTF-8
xdg-user-dirs-gtk-update
echo 选“不要再提醒”和“保持原来的名称”
```

`该配置需要重启生效`

### 2.6、ssh忽略known_hosts

```sh
mkdir -p ~/.ssh
echo -e "StrictHostKeyChecking no\nUserKnownHostsFile /dev/null" > ~/.ssh/config
```

### 2.7、安装gofont字体

从github下载gofont, `git clone https://github.com/golang/image.git`, 拷贝`font/gofont/ttfs/`下面的字体到系统字体中:

```sh
sudo mkdir /usr/share/fonts/gofonts
sudo cp font/gofont/ttfs /usr/share/fonts/gofonts
cd /usr/share/fonts/gofonts
sudo chmod 744 *
sudo mkfontscale
sudo mkfontdir
sudo fc-cache -f -v
```

接着注销ubuntu，再次登录就可以使用该go字体了。

## 3、安装有道字典

安装有道词典的[1.0.2版本](http://codown.youdao.com/cidian/linux/youdao-dict_1.0.2~ubuntu_amd64.deb)，据说1.1.0版本有依赖问题。暂时不要挑战了。

```sh
   zero@zero:/media/zero/Myself/Ubuntu1604$ sudo dpkg -i youdao-dict_1.0.2~ubuntu_amd64.deb
     正在选中未选择的软件包 youdao-dict。
     (正在读取数据库 ... 系统当前共安装有 218440 个文件和目录。)
     正准备解包 youdao-dict_1.0.2~ubuntu_amd64.deb  ...
     正在解包 youdao-dict (1.0.2~ubuntu) ...
     dpkg: 依赖关系问题使得 youdao-dict 的配置工作不能继续:
      youdao-dict 依赖于 python3-pyqt5；然而:
       未安装软件包 python3-pyqt5。
      youdao-dict 依赖于 python3-xlib；然而:
       未安装软件包 python3-xlib。
      youdao-dict 依赖于 tesseract-ocr；然而:
       未安装软件包 tesseract-ocr。
      youdao-dict 依赖于 tesseract-ocr-eng；然而:
       未安装软件包 tesseract-ocr-eng。
      youdao-dict 依赖于 tesseract-ocr-chi-sim；然而:
       未安装软件包 tesseract-ocr-chi-sim。
      youdao-dict 依赖于 tesseract-ocr-chi-tra；然而:
       未安装软件包 tesseract-ocr-chi-tra。
      youdao-dict 依赖于 ttf-wqy-microhei；然而:
       未安装软件包 ttf-wqy-microhei。
      youdao-dict 依赖于 python3-pyqt5.qtmultimedia；然而:
       未安装软件包 python3-pyqt5.qtmultimedia。
      youdao-dict 依赖于 python3-pyqt5.qtquick；然而:
       未安装软件包 python3-pyqt5.qtquick。
      youdao-dict 依赖于 python3-pyqt5.qtwebkit；然而:
       未安装软件包 python3
     dpkg: 处理软件包 youdao-dict (--install)时出错:
      依赖关系问题 - 仍未被配置
     正在处理用于 hicolor-icon-theme (0.15-0ubuntu1) 的触发器 ...
     正在处理用于 gnome-menus (3.13.3-6ubuntu3.1) 的触发器 ...
     正在处理用于 desktop-file-utils (0.22-1ubuntu5.1) 的触发器 ...
     正在处理用于 bamfdaemon (0.5.3~bzr0+16.04.20160824-0ubuntu1) 的触发器 ...
     Rebuilding /usr/share/applications/bamf-2.index...
     正在处理用于 mime-support (3.59ubuntu1) 的触发器 ...
     在处理时有错误发生:
      youdao-dict
  zero@zero:/media/zero/Myself/Ubuntu1604$ sudo apt-get install python3-pyqt5
  正在读取软件包列表... 完成
  正在分析软件包的依赖关系树
  正在读取状态信息... 完成
  您可能需要运行“apt-get -f install”来纠正下列错误:
  下列软件包有未满足的依赖关系:
   python3-pyqt5 : 依赖: libqt5designer5 (>= 5.0.2) 但是它将不会被安装
                   依赖: libqt5help5 (>= 5.3.0) 但是它将不会被安装
                   依赖: sip-py3api-11.2
   youdao-dict : 依赖: python3-xlib 但是它将不会被安装
                 依赖: tesseract-ocr 但是它将不会被安装
                 依赖: tesseract-ocr-eng 但是它将不会被安装
                 依赖: tesseract-ocr-chi-sim 但是它将不会被安装
                 依赖: tesseract-ocr-chi-tra 但是它将不会被安装
                 依赖: ttf-wqy-microhei
                 依赖: python3-pyqt5.qtmultimedia 但是它将不会被安装
                 依赖: python3-pyqt5.qtquick 但是它将不会被安装
                 依赖: python3-pyqt5.qtwebkit 但是它将不会被安装
                 依赖: qtdeclarative5-controls-plugin 但是它将不会被安装
  E: 有未能满足的依赖关系。请尝试不指明软件包的名字来运行“apt-get -f install”(也可以指定一个解决办法)。
  zero@zero:/media/zero/Myself/Ubuntu1604$ sudo apt-get -f install
  正在读取软件包列表... 完成
  正在分析软件包的依赖关系树
  正在读取状态信息... 完成
  正在修复依赖关系... 完成
  将会同时安装下列软件:
    fonts-wqy-microhei liblept5 libopenjp2-7 libqt5clucene5 libqt5designer5
    libqt5help5 libqt5multimediawidgets5 libtesseract3 python3-pyqt5
    python3-pyqt5.qtmultimedia python3-pyqt5.qtquick python3-pyqt5.qtwebkit
    python3-sip python3-xlib qml-module-qtquick-controls
    qtdeclarative5-controls-plugin tesseract-ocr tesseract-ocr-chi-sim
    tesseract-ocr-chi-tra tesseract-ocr-eng tesseract-ocr-equ tesseract-ocr-osd
    ttf-wqy-microhei
  建议安装:
    python3-pyqt5-dbg
  下列【新】软件包将被安装:
    fonts-wqy-microhei liblept5 libopenjp2-7 libqt5clucene5 libqt5designer5
    libqt5help5 libqt5multimediawidgets5 libtesseract3 python3-pyqt5
    python3-pyqt5.qtmultimedia python3-pyqt5.qtquick python3-pyqt5.qtwebkit
    python3-sip python3-xlib qml-module-qtquick-controls
    qtdeclarative5-controls-plugin tesseract-ocr tesseract-ocr-chi-sim
    tesseract-ocr-chi-tra tesseract-ocr-eng tesseract-ocr-equ tesseract-ocr-osd
    ttf-wqy-microhei
  升级了 0 个软件包，新安装了 23 个软件包，要卸载 0 个软件包，有 1 个软件包未被升级。
  有 1 个软件包没有被完全安装或卸载。
  需要下载 55.1 MB 的归档。
  解压缩后会消耗 191 MB 的额外空间。
  您希望继续执行吗？ [Y/n] y
```

## 4、安装输入法

### 4.1、搜狗输入发

参照[Ubuntu 16.04安装sogou 拼音输入法](http://blog.csdn.net/ljheee/article/details/52966456)和[Ubuntu16.04常用软件](http://blog.csdn.net/bt8023cpt/article/details/53997607)

下载[搜狗输入法](http://pinyin.sogou.com/)

```bash
$ cd download/
$ sudo dpkg -i sogoupinyin_2.1.0.0082_amd64.deb
$ sudo apt-get install -f
$ sudo dpkg -i sogoupinyin_2.1.0.0082_amd64.deb
```

## 5、安装播放器

### 5.1、网易云音乐

下载[网易云音乐](http://music.163.com/#/download)

```sh
$ cd download/
$ sudo dpkg -i netease-cloud-music_1.1.0_amd64_ubuntu.deb
$ sudo apt-get install -f
$ sudo dpkg -i netease-cloud-music_1.1.0_amd64_ubuntu.deb
```

## 6、安装开发工具

* [安装激活goland2017.3版本](2017-12-05-active-goland-2017.3.md)
* [安装LiteIDE](https://golangtc.com/download/liteide)

```sh
    tar jxvf liteidex32.1.linux64-qt4.tar.bz2
    cd liteide
    编辑liteide.desktop,根据自己的路径修改exec和icon配置
    然后拷贝该配置到$HOME/.local/share/applications下
    cd $HOME/liteide/bin/
    启动liteide
    并锁定到启动器上即可。
```

* [安装vscode指引](https://code.visualstudio.com/docs/setup/linux)
* [安装atom](http://www.linuxidc.com/Linux/2014-05/101501.htm)
* [安装typora](https://www.typora.io/#linux)

```sh
    # optional, but recommended
    sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys BA300B7755AFCFAE
    # add Typora's repository
    sudo add-apt-repository 'deb http://typora.io linux/'
    sudo apt-get update
    # install typora
    sudo apt-get install typora
```

```sh
    sudo add-apt-repository ppa:webupd8team/atom
    sudo apt-get update
    sudo apt-get install atom
```

## 7、安装xmind

* [下载xmind](http://www.xmind.net/download/linux/)
* [安装xmind](http://blog.csdn.net/faryang/article/details/70238326)

## 8、安装FoxitReader

* [下载FoxitReader x64版本](http://cdn01.foxitsoftware.com/pub/foxit/reader/desktop/linux/2.x/2.4/en_us/FoxitReader2.4.1.0609_Server_x64_enu_Setup.run.tar.gz)
* [安装FoxitReader](https://www.cnblogs.com/EasonJim/p/7231936.html)

## 9、安装WPS

* 下载[WPS软件包](http://linux.wps.cn/)
* 下载[WPS依赖的字体](https://github.com/neujie/www.neujie.cn/blob/master/packages/wps_symbol_fonts.zip)(违删)
* 安装字体方法参照`安装gofont操作`

## 10、图形软件

### 10.1、画图工具
#### 10.1.1、Dia

在软件仓库里找`dia`或者使用:

```bash
sudo apt-get install dia
```

Dia是基于GTK的开源流程图软件，适用于Linux, Unix和Windows，以 GPL 许可发布。一个10MB量级的免费软件，代替visio基本没问题。发现直接在dia中无法用fcitx输入中文

解决办法:
* <1>在命令行中用：dia-normal来运行dia，而不要用默认的dia来运行。这样就可以输入中文啦，不过问题就是toolbar会独立出来。

* <2> 修改dia的启动指令

打开软件后发现不能输入中文，网上搜索一圈后找到以下解决方案

    sudo vim /usr/bin/dia

然后把

    dia-normal --integrated "$@"

修改成

    dia-normal  "$@"

重新启动dia，输入法选择系统默认（我的系统输入法是搜狗输入法(fcitx)），就可以输入中文了。
不过这样一改，工具条变成浮动的了。



#### 10.1.2、kolourpaint

```bash
sudo apt-get install  kolourpaint4
```

#### 10.1.3、shutter

```bash
sudo apt-get install shutter
```

#### 10.1.4、StarUML

* 下载StarUML:

  [官网地址libgcrypt11_1.5.3-2ubuntu4.4_amd64.deb](https://launchpad.net/~ubuntu-security-proposed/+archive/ubuntu/ppa/+build/10628783)

* 安装StarUML:
  * 下载并安装依赖库[libgcrypt11_1.5.3-2ubuntu4.4_amd64.deb](https://launchpad.net/~ubuntu-security-proposed/+archive/ubuntu/ppa/+build/10628783)

```sh
  zero@zero ~/Downloads> sudo dpkg -i libgcrypt11_1.5.3-2ubuntu4.4_amd64.deb
```

  * 安装StarUML

```sh
    zero@zero ~/Downloads> sudo dpkg -i StarUML-v2.8.0-64-bit.deb
```


## 11、挂在其他磁盘

### 11.1、查看其他磁盘卷

```sh
zero@zero:~$ sudo fdisk -l
[sudo] zero 的密码:
Disk /dev/sda: 232.9 GiB, 250059350016 bytes, 488397168 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xe159e159

设备       启动     Start    末尾    扇区   Size Id 类型
/dev/sda1  *       206848 102606847 102400000  48.8G  7 HPFS/NTFS/exFAT
/dev/sda2       102608894 205006847 102397954  48.8G  5 扩展
/dev/sda3       205006848 488394751 283387904 135.1G  7 HPFS/NTFS/exFAT
/dev/sda5       102608896 110606335   7997440   3.8G 82 Linux 交换 / Solaris
/dev/sda6       110608384 205006847  94398464    45G 83 Linux

Partition table entries are not in disk order.
```

### 11.2、挂在操作
识别出来哪个是自己的盘符，然后创建自己可以识别的名称，然后进行挂载操作即可。

```sh
zero@zero:~$ sudo mkdir /mnt/C
zero@zero:~$ sudo mount -t ntfs /dev/sda1 /mnt/C -o iocharset=utf8,umask=0
zero@zero:~$ sudo mkdir /mnt/E
zero@zero:~$ sudo mount -t ntfs /dev/sda3 /mnt/E -o iocharset=utf8,umask=0
```

### 11.3、持久化，写入系统配置中
挂载磁盘的位置在:`/etc/fstab`

```sh
zero@zero:~$ cat /etc/fstab
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/sda6 during installation
UUID=6acca2e8-0890-409d-b742-52b3ebac2050 /               ext4    errors=remount-ro 0       1
# swap was on /dev/sda5 during installation
UUID=16bc804c-af93-49da-820e-269c7a09d945 none            swap    sw              0       0
```

加入我们的指令:

```sh
zero@zero:~$ cat /etc/fstab
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/sda6 during installation
UUID=6acca2e8-0890-409d-b742-52b3ebac2050 /               ext4    errors=remount-ro 0       1
# swap was on /dev/sda5 during installation
UUID=16bc804c-af93-49da-820e-269c7a09d945 none            swap    sw              0       0
# mount -t ntfs /dev/sda1 /mnt/C -o iocharset=utf8,umask=0
/dev/sda1 /mnt/C ntfs utf8,umask=0
# mount -t ntfs /dev/sda3 /mnt/E -o iocharset=utf8,umask=0
/dev/sda3 /mnt/E ntfs utf8,umask=0
```

## 12、安装mac主题

### 添加macbuntu资源

    $sudo add-apt-repository ppa:noobslab/macbuntu

### 安装mac窗口主题

    $sudo apt-get install  macbuntu-os-ithemes-lts-v7

### 安装mac图标主题
    $sudo apt-get install  macbuntu-os-icons-lts-v7

### 若想卸载，卸载指令

    cd /usr/share/icons/mac-cursors && sudo ./uninstall-mac-cursors.sh
    sudo apt-get remove macbuntu-os-icons-lts-v7 macbuntu-os-ithemes-lts-v7

### 安装 Slingscold（替代Launchpad）

    sudo add-apt-repository ppa:noobslab/macbuntu
    sudo apt-get update
    sudo apt-get install slingscold

### 安装Albert Spotlight (替代 Mac Spotlight)

    sudo add-apt-repository ppa:noobslab/macbuntu
    sudo apt-get update
    sudo apt-get install albert

### 安装 Plank Dock

    sudo apt-get install plank

### 安装 Plank 主题:

    sudo add-apt-repository ppa:noobslab/macbuntu
    sudo apt-get update
    sudo apt-get install macbuntu-os-plank-theme-lts-v7

    安装 Ctrl + Right Click 选择主题。

### 卸载主题命令:

    sudo apt-get autoremove plank macbuntu-os-plank-theme-lts-v7

### 将面板上的 ‘Ubuntu Desktop’ 文字替换 ‘Mac’

    cd && wget -O Mac.po http://drive.noobslab.com/data/Mac/change-name-on-panel/mac.po
    cd /usr/share/locale/en/LC_MESSAGES; sudo msgfmt -o unity.mo ~/Mac.po;rm ~/Mac.po;cd

    语言上如果是其他非英语语言，将 /en 做相应修改:



### 恢复原来的桌面文字命令:

    cd && wget -O Ubuntu.po http://drive.noobslab.com/data/Mac/change-name-on-panel/ubuntu.po
    cd /usr/share/locale/en/LC_MESSAGES; sudo msgfmt -o unity.mo ~/Ubuntu.po;rm ~/Ubuntu.po;cd

### 启动器上苹果Logo
#### 命令:
    wget -O launcher_bfb.png http://drive.noobslab.com/data/Mac/launcher-logo/apple/launcher_bfb.png
    sudo mv launcher_bfb.png /usr/share/unity/icons/

#### 恢复图标:

    wget -O launcher_bfb.png http://drive.noobslab.com/data/Mac/launcher-logo/ubuntu/launcher_bfb.png
    sudo mv launcher_bfb.png /usr/share/unity/icons/



### 安装unity-tweak-tools配置主题工具
    $sudo apt-get install  unity-tweak-tools
    $sudo apt-get install gnome-tweak-tool

### 配置 Mac 字体:

    安装字体命令:
    wget -O mac-fonts.zip http://drive.noobslab.com/data/Mac/macfonts.zip
    sudo unzip mac-fonts.zip -d /usr/share/fonts; rm mac-fonts.zip
    sudo fc-cache -f -v

    使用 Unity-Tweak-Tool, Gnome-Tweak-Tool 或 Ubuntu Tweak 软件更换字体。


### 修改启动界面:

    sudo add-apt-repository ppa:noobslab/themes
    sudo apt-get update
    sudo apt-get install macbuntu-os-bscreen-lts-v7

    如果你喜欢 MBuntu 启动界面，你想恢复到 Ubuntu ，使用命令:
    sudo apt-get autoremove macbuntu-os-bscreen-lts-v7

### 登陆迎宾界面:

#### 安装命令:
注意安装这个东东有风险，如果你不是在意，建议不安装，安装这个仅仅适合于 Ubuntu Unity – Lightdm 和 Ubuntu Gnome – gdm，其他桌面环境有风险。

    sudo add-apt-repository ppa:noobslab/themes
    sudo apt-get update
    sudo apt-get install macbuntu-os-lightdm-lts-v7

#### 卸载恢复命令:

    sudo apt-get remove macbuntu-os-lightdm-lts-v7



## 90、其他操作

### 90.1、软件卸载

#### 卸载ubutnu自带的vi

    sudo apt-get remove vim-common

#### 自动卸载一些软件包，比如内核模块

    sudo apt autoremove

### 90.2、通讯工具
* [微信&QQ工具](http://blog.csdn.net/zhangrelay/article/details/52398037)
* [linux版本的wechat](http://blog.csdn.net/minione_2016/article/details/53289430?locationNum=4&fps=1)

### 90.3、其他链接
* [Ubuntu 16.04 LTS安装好需要设置的15件事](http://www.linuxidc.com/Linux/2016-04/130519.htm)
* [Ubuntu 安装terminator 后更改默认终端设置](http://blog.csdn.net/qq_20336817/article/details/46122849)

```sh
$ sudo apt-get install terminator
$ sudo apt-get install dconf-tools

若想换回terminal
    exec  gnome-terminal
    exec-arg -x
否则:
    exec  x-terminal-emulator
    exec-arg -e
```
