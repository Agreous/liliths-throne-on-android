# 莉莉丝的王座安卓手机运行教程(施工中)
### 前言
由于莉莉丝的王座是使用JAVA进行开发的游戏，并不能方便的移植到移动端基于安卓的设备上。因此本文将通过使用termux安装proot linux容器的方式来曲线实现在安卓手机上游玩莉莉丝的王座。

### 警告
因为我们需要在手机上额外跑一个近乎完整的linux桌面环境，因此需要有至少6GB的系统内存和至少10GB的储存空间才能完成这一教程。同时，因为本教程采用无需root的proot容器方案，会对linux容器的运行造成一定的性能损失，以及无法使用大部分的图形加速能力。

### 安装应用程序
为了安装proot容器和linux桌面环境并实现一键启动，我们需要安装以下app：[termux](https://github.com/termux/termux-app),[termux-x11](https://github.com/termux/termux-x11),[termux-api](https://github.com/termux/termux-api),[termux-widget](https://github.com/termux/termux-widget)。点击链接进入对应github页面之后下载releases中最新发布版本，对于termux,选择后缀带有universal的安装包，对于termux-x11,进入github页面后点击Actions页面选择最新build下载带有universal后缀的压缩包解压得到安装包。

## 初步设置
打开termux之后，下滑通知栏，点击termux通知中的`ACQUIRE WAKELOCK`按钮使termux可以在后台运行。
![Screenshot_20231110-153646_Termux](https://github.com/Agreous/liliths-throne-on-android/assets/46571579/dffd57db-6d3d-4fc2-90e4-bff5baa46053)

之后在termux中输入`termux-change-repo`命令对termux的默认软件源进行替换。输入命令后会进入一个伪图形界面，第一步直接按回车跳过，第二部用termux工具栏提供的方向键向下移动到需要的镜像源后按空格选择，按回车继续。随后输入`pkg install x11-repo root-repo tur-repo`安装需要的软件源。
![Screenshot_20231110-153716_Termux](https://github.com/Agreous/liliths-throne-on-android/assets/46571579/337d9b84-2c3b-4192-a5d2-6f7a9c0d2daa)
![Screenshot_20231110-165021_Termux](https://github.com/Agreous/liliths-throne-on-android/assets/46571579/a8a49d85-5f39-4183-a5f2-f8682691407e)


安装好软件源之后再次输入`termux-change-repo`命令对新安装的软件源进行换源，如图所示
![Screenshot_20231110-154006_Termux](https://github.com/Agreous/liliths-throne-on-android/assets/46571579/e6f16bda-e6a6-4678-88f2-c4ec9748e54b)
![Screenshot_20231110-154023_Termux](https://github.com/Agreous/liliths-throne-on-android/assets/46571579/f9a37419-07ed-4196-bd35-3562caea9b19)




最后输入`pkg install proot-distro pulseaudio vim virglrenderer-android termux-x11-nightly`安装所需的termux软件包。

### 安装linux容器
输入以下命令安装一个arch linux的proot容器，你可以使用`proot-distro list`命令来查看所有可用的发行版，但是本文只会以debian作为教程基础。
（在进行这一步的时候可能需要挂梯子）

```
proot-distro install debian
```

安装后使用以下命令登入容器，--user参数表示登录指定帐户，在初始状态下仅有root。--shared-tmp则是將Termux的tmp目录挂载到proot內部以共享X服务器资源（之后会用到，这里可以不加）。

```
proot-distro login debian --user root --shared-tmp
```

### 切换软件源，更新系统并进行linux基本设置
#### 切换软件源
在进入linux系统之后，我们要做的第一件事就是将系统软件源更换为国内镜像源以方便后续软件安装，用系统自带的vim对`/etc/apt/sources.list`进行编辑。

```
vim /etc/apt/sources.list
```

按i进入编辑模式，将所有内容删除，替换为

```
deb http://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm main contrib non-free non-free-firmware
# deb-src http://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm main contrib non-free non-free-firmware

deb http://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm-updates main contrib non-free non-free-firmware
# deb-src http://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm-updates main contrib non-free non-free-firmware

deb http://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm-backports main contrib non-free non-free-firmware
# deb-src http://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm-backports main contrib non-free non-free-firmware

deb http://mirrors.tuna.tsinghua.edu.cn/debian-security bookworm-security main contrib non-free non-free-firmware
# deb-src http://mirrors.tuna.tsinghua.edu.cn/debian-security bookworm-security main contrib non-free non-free-firmware
```
然后按esc退出编辑模式，输入:wq保存退出，最后输入`apt update`刷新软件源完成工作。
具体软件源设置参考[清华大学源](https://mirrors.tuna.tsinghua.edu.cn/help/debian/)的帮助

#### 更新系统并进行linux基本设置
输入`apt upgrade`更新系统，完成后开始对linux系统进行基本设置:

1.为root用户设置密码
```
passwd
```
2.增加普通用户，并为普通用户设置用户组，并设置密码
```
apt install sudo vim
groupadd storage
groupadd wheel
groupadd video
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
注意，此处以及下文中的所有user都可以替换为你自己的用户名。
### 安装所需工具与桌面环境
1.安装中文字体，输入法与一些工具

```
sudo apt install networkmanager pulseaudio noto-fonts-cjk git fcitx5* locales
```

2.设定系统时区与语言，输入法

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
如果此步完成后语言仍未设置成中文，则输入`source /etc/locale.conf`,可以用`apt`命令的输出作为参考。
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
sudo apt install kde-standard
```

安装xfce
```
sudo apt install xfce4 xfce4-goodies
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


proot-distro login debian --user user --shared-tmp -- bash -c "export DISPLAY=:0 PULSE_SERVER=tcp:127.0.0.1; dbus-launch --exit-with-session startplasma-x11"
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


proot-distro login debian --user user --shared-tmp -- bash -c "export DISPLAY=:0 PULSE_SERVER=tcp:127.0.0.1; dbus-launch --exit-with-session xfce4-session"
```
然后给脚本赋予执行权限
```
chmod +x .shortcuts/startproot.sh
```
最后在手机桌面添加Termux Widget提供的小部件，点击一下小部件右上角的刷新按钮就可以使用一键启动脚本了。

## 安装JAVA环境
debian的软件源中有openjdk,因此可以直接安装。
输入`sudo apt install openjdk-17-jdk`即可安装。


## 关于游戏启动
启动游戏的方法很简单，在游戏根目录打开终端，输入`java -jar 游戏主文件.jar`就可以启动游戏，但需要注意的是`java -jar`这一命令对文件名有着较为严格的要求，在使用前需要将游戏主文件文件名内的空格和特殊字符删除。

## 如何在手机与proot容器之间共享文件
下载并安装[MT管理器](https://mt2.cn/download/)，在termux程序下输入`termux-setup-storage`,对给出的任何提示无脑按y，之后打开MT管理器，点击程序主界面左上角的三条横线就可以看到Termux Home选项卡了。我们安装的proot容器文件位于该文件夹内的/installed-rootfs/archlinux文件夹当中。

## 最终效果
![Screenshot_2023-10-13-13-50-11-539_com termux x11](https://github.com/Agreous/liliths-throne-on-android/assets/46571579/46424a9c-b26c-42f1-b61b-c6c195bb7726)

### FAQ

#### 那么，该从哪里获取aarch64版本的游戏呢
[sir,this way](https://github.com/CKRainbow/liliths-throne-localization/tags)

或者你也可以自己编译，但那就是另一段故事了

#### 我看作者提供的jar版本游戏是全平台通用的啊，为什么到你这里就需要分平台了呢
因为作者提供的jar版本游戏不包含任何依赖库文件，而依赖之一javafx在很多平台的高版本openjdk中并不包含，就算是提供了openjfx(和javafx是一样的)包的平台通常也因为分包安装导致java无法直接调用需要的库文件，导致运行作者给出的jar文件通常会报错。为游戏打包依赖库文件可以解决这个问题，但是就需要根据目标平台来打包所需的依赖库文件了。因此就失去了平台通用性。

#### 之前的教程是以arch linux为基础的，为什么要换成debian呢
因为大费周章安装一个linux容器就为了玩一个黄油实在是太浪费了，日后你还可以参照其他教程为这个容器安装box86/box64和wine来玩更多黄油，而arm版的arch linux不能提供32位的库文件，会对之后安装这些东西造成阻碍。因此我更换了教程的发行版。

## 参考资料
[ivon's blog](https://ivonblog.com/posts/termux-proot-distro-debian/)

[参考安装视频](https://www.bilibili.com/video/BV1we4y1p73H)
