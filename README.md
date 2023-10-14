# 莉莉丝的王座安卓手机运行教程(施工中)
### 前言
由于莉莉丝的王座是使用JAVA进行开发的游戏，在一开始就没有考虑过可移植性，因此本文将通过使用termux安装proot linux容器的方式来曲线实现在安卓手机上游玩莉莉丝的王座。

### 警告
因为我们需要在手机上额外跑一个近乎完整的linux桌面环境，因此需要有至少6GB的系统内存和至少10GB的储存空间才能完成这一教程。同时，因为本教程采用无需root的proot容器方案，会对linux容器的运行造成一定的性能损失，以及无法使用大部分的图形加速能力。

### 安装应用程序
为了安装proot容器和linux桌面环境并实现一键启动，我们需要安装以下app：[termux](https://github.com/termux/termux-app),[termux-x11](https://github.com/termux/termux-x11),[termux-api](https://github.com/termux/termux-api),[termux-widget](https://github.com/termux/termux-widget)。点击链接进入对应github页面之后下载releases中最新发布版本，对于termux,选择后缀带有universal的安装包，对于termux-x11,进入github页面后点击Actions页面选择最新build下载带有universal后缀的压缩包解压得到安装包。

## 初步设置
打开termux之后，下滑通知栏，点击termux通知中的`ACQUIRE WAKELOCK`按钮使termux可以在后台运行。之后在termux中输入`termux-change-repo`命令对termux的默认软件源进行替换。输入命令后会进入一个伪图形界面，第一步直接按回车跳过，第二部用termux工具栏提供的方向键向下移动到mirrors of china选项后按空格选择，按回车继续。随后输入以下命令执行以安装后续所需的软件包和更新系统。

```
pkg upgrade
pkg install proot-distro pulseaudio vim virglrenderer-android x11-repo termux-x11-nightly
```
### 安装linux容器
输入以下命令安装一个arch linux的proot容器，你可以使用`proot-distro list`命令来查看所有可用的发行版，但是本文只会以arch linux作为教程基础。

```
proot-distro install archlinux
```

安装后使用以下命令登入容器，--user参数表示登录指定帐户，在初始状态下仅有root。--shared-tmp则是將Termux的tmp目录挂载到proot內部以共享X服务器资源（之后会用到，这里可以不加）。

```
proot-distro login archlinux --user root --shared-tmp
```

### 切换软件源，更新系统并进行linux基本设置
#### 切换软件源
在进入linux系统之后，我们要做的第一件事就是将系统软件源更换为国内镜像源以方便后续软件安装，用系统自带的vim对`/etc/pacman.d/mirrorlist`进行编辑。

```
vim /etc/pacman.d/mirrorlist
```

找到唯一一个没有被#注释的Server=行，按i进入编辑模式，将其替换为

```
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxarm/$arch/$repo
```
然后按esc退出编辑模式，输入:wq保存退出，最后输入`pacman -Syy`刷新软件源完成工作。

#### 更新系统并进行linux基本设置
输入`pacman -Syu`更新系统，完成后开始对linux系统进行基本设置:

1.为root用户设置密码
```
passwd
```
2.增加普通用户，并为普通用户设置用户组，并设置密码
```
pacman -S sudo vim
useradd -m -g users -G wheel,audio,video,storage -s /bin/bash user
passwd user
```
在linux中root用户拥有着系统的最高权限，因此我们不能时刻用root账户进行登录，同时我们要为之后安装桌面环境对用户组进行设置，其中的`user`可以替换为你想要的用户名。

3.赋予普通用户使用root权限的能力，编辑`/etc/sudoers`

```
vim /etc/sudoers

在root ALL=(ALL) ALL的下一行加入以下內容:

user ALL=(ALL) ALL
```
最后就可以切换到普通用户账号了，之后都是用这个普通用户账号进行操作。

```
su user
```

### 安装所需工具与桌面环境
1.安装中文字体，SSH,输入法与一些工具

```
sudo pacman -S networkmanager xorg xorg-server pulseaudio noto-fonts git openssh fakeroot base-devel fcitx5-config-qt fcitx5-qt fcitx5-gtk fcitx5-chinese-addons libpinyin noto-color-emoji p7zip unrar firefox
```

2.（可选）安装AUR助手yay

AUR是arch linux的用户软件源，可以从里面搞到很多有用的软件，不过我们运行莉莉丝的王座不需要用到它，因此是可选的。

```
sudo pacman -S --needed git base-devel && git clone https://aur.archlinux.org/yay.git && cd yay && makepkg -si
```

3.设定系统时区与语言，输入法

输入以下命令将系统时区设置为上海（GMT+8）

```
sudo ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```
然后编辑`/etc/locale.gen`文件，找到其中被注释的`#zh_CN.UTF-8`，取消注释。

输入以下命令生成语言设定
```
sudo locale-gen
sudo echo "LANG=zh_CN.UTF-8" >> /etc/locale.conf
```
最后编辑`/.profile`，在文件尾添加以下内容
```
fcitx5 &
GTK_IM_MODULE=fcitx
QT_IM_MODULE=fcitx
XMODIFIERS=@im=fcitx
SDL_IM_MODULE=fcitx
GLFW_IM_MODULE=ibus
```

4.安装桌面环境

由于proot容器缺少systemd，gnome无法启动，因此我们可选的常用桌面环境仅剩下KDE和xfce,KDE美观性更好，但是更吃资源，并且动画在软件渲染下表现不佳。因此也可选用更轻量的xfce。下面会给出两种桌面环境的安装命令，按需选择。

安装KDE
```
sudo pacman -S plasma sddm dolphin ark konsole kate dolphin-plugins elisa gwenview
```

安装xfce
```
sudo pacman -S xfce4 xfce4-goodies lightdm
```

## 设置一键启动
首先需要确保[termux-api](https://github.com/termux/termux-api)和[termux-widget](https://github.com/termux/)都已被安装，随后赋予termux允许显示在其他应用上层权限。然后在termux程序内（注意，不是proot容器内，如果你在proot容器内，输入exit命令退出proot容器）输入以下命令创建用于一键启动的脚本
```
mkdir .shortcuts
vim .shortcuts/startproot.sh
```
随后根据前面选择安装的桌面环境选择下面两个内容之一填入脚本当中

使用KDE
```
#!/bin/bash


killall -9 termux-x11 Xwayland pulseaudio virgl_test_server_android termux-wake-lock


termux-toast "Starting X11"
am start --user 0 -n com.termux.x11/com.termux.x11.MainActivity
XDG_RUNTIME_DIR=${TMPDIR}
termux-x11 :0 -ac &
sleep 3


pulseaudio --start --load="module-native-protocol-tcp auth-ip-acl=127.0.0.1 auth-anonymous=1" --exit-idle-time=-1
pacmd load-module module-native-protocol-tcp auth-ip-acl=127.0.0.1 auth-anonymous=1


virgl_test_server_android &


proot-distro login archlinux --user user --shared-tmp -- bash -c "export DISPLAY=:0 PULSE_SERVER=tcp:127.0.0.1; dbus-launch --exit-with-session startplasma-x11"
```

使用xfce
```
#!/bin/bash


killall -9 termux-x11 Xwayland pulseaudio virgl_test_server_android termux-wake-lock


termux-toast "Starting X11"
am start --user 0 -n com.termux.x11/com.termux.x11.MainActivity
XDG_RUNTIME_DIR=${TMPDIR}
termux-x11 :0 -ac &
sleep 3


pulseaudio --start --load="module-native-protocol-tcp auth-ip-acl=127.0.0.1 auth-anonymous=1" --exit-idle-time=-1
pacmd load-module module-native-protocol-tcp auth-ip-acl=127.0.0.1 auth-anonymous=1


virgl_test_server_android &


proot-distro login archlinux --user user --shared-tmp -- bash -c "export DISPLAY=:0 PULSE_SERVER=tcp:127.0.0.1; dbus-launch --exit-with-session startxfce4"
```
然后给脚本赋予执行权限
```
chmod +x .shortcuts/startproot.sh
```
最后在手机桌面添加Termux Widget提供的小部件，点击一下小部件右上角的刷新按钮就可以使用一键启动脚本了。

## 安装JAVA环境
我们在之前的步骤里面已经安装好了linux容器的桌面环境，之后就可以全程在linux容器中进行操作。

1.下载JAVA

由于arch并没有在软件源中提供arm版本的java以供安装，所以我们需要手动下载并设置JAVA环境，打开[微软打包版本的openjdk](https://learn.microsoft.com/zh-cn/java/openjdk/download)，找到提供的最新版本中的linux aarch64版本，下载类型为tar.gz的文件，下载完成后将其解压，并放在一个合适的位置。

2.设置JAVA环境变量

打开`/etc/profile`
```
vim /etc/profile
```
在文件末尾添加以下内容
```
export JAVA_HOME=这里替换成你存放刚才解压的JAVA的路径
export JRE_HOME=${JAVA_HOMR}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
```
保存退出后输入`source /etc/profile`重载文件以应用环境变量。随后输入`java --version`以确认环境变量是否起效。

## 关于游戏启动
启动游戏的方法很简单，在游戏根目录打开终端，输入`java -jar 游戏主文件.jar`就可以启动游戏，但需要注意的是`java -jar`这一命令对文件名有着较为严格的要求，在使用前需要将游戏主文件文件名内的空格和特殊字符删除。

## 如何在手机与proot容器之间共享文件
下载并安装[MT管理器](https://mt2.cn/download/)，在termux程序下输入`termux-setup-storage`,对给出的任何提示无脑按y，之后打开MT管理器，点击程序主界面左上角的三条横线就可以看到Termux Home选项卡了。我们安装的proot容器文件位于该文件夹内的/installed-rootfs/archlinux文件夹当中。
## 参考资料
[ivon's blog](https://ivonblog.com/posts/termux-proot-distro-archlinux/#5-%E5%AE%89%E8%A3%9D%E6%A1%8C%E9%9D%A2%E7%92%B0%E5%A2%83%E5%92%8C%E5%B8%B8%E7%94%A8%E5%B7%A5%E5%85%B7)