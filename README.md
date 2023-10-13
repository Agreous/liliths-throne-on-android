# 莉莉丝的王座安卓手机运行教程(施工中)
前言
由于莉莉丝的王座是使用JAVA进行开发的游戏，在一开始就没有考虑过可移植性，因此本文将通过使用termux安装proot linux容器的方式来曲线实现在安卓手机上游玩莉莉丝的王座。

警告
因为我们需要在手机上额外跑一个近乎完整的linux桌面环境，因此需要有至少6GB的系统内存和至少10GB的储存空间才能完成这一教程。

安装应用程序
为了安装proot容器和linux桌面环境并实现一键启动，我们需要安装以下app：[termux](https://github.com/termux/termux-app),[termux-x11](https://github.com/termux/termux-x11),[termux-api](https://github.com/termux/termux-api),[termux-widget](https://github.com/termux/termux-widget)。点击链接进入对应github页面之后下载releases中最新发布版本，对于termux,选择后缀带有universal的安装包，对于termux-x11,进入github页面后点击Actions页面选择最新build下载带有universal后缀的压缩包解压得到安装包。

## 初步设置
打开termux之后，下滑通知栏，点击termux通知中的`ACQUIRE WAKELOCK`按钮使termux可以在后台运行。之后在termux中输入`termux-change-repo`命令对termux的默认软件源进行替换。输入命令后会进入一个伪图形界面，第一步直接按回车跳过，第二部用termux工具栏提供的方向键向下移动到mirrors of china选项后按空格选择，按回车继续。随后输入以下命令执行以安装后续所需的软件包和更新系统。

```
pkg upgrade
pkg install proot-distro pulseaudio vim virglrenderer-android
```
###安装linux容器
输入以下命令安装一个arch linux的proot容器，你可以使用`proot-distro list`命令来查看所有可用的发行版，但是本文只会以arch linux作为教程基础。

```
proot-distro install archlinux
```

安装后使用以下命令登入容器，--user参数表示登录指定帐户，在初始状态下仅有root。--shared-tmp则是將Termux的tmp目录挂载到proot內部以共享X服务器资源（之后会用到，这里可以不加）。

```
proot-distro login archlinux --user root --shared-tmp
```

###切换软件源，更新系统并进行linux基本设置
####切换软件源
在进入linux系统之后，我们要做的第一件事就是将系统软件源更换为国内镜像源以方便后续软件安装，用系统自带的vim对`/etc/pacman.d/mirrorlist`进行编辑。

```
vim /etc/pacman.d/mirrorlist
```

找到唯一一个没有被#注释的Server=行，按i进入编辑模式，将其替换为

```
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxarm/$arch/$repo
```
然后按esc退出编辑模式，输入:wq保存退出，最后输入`pacman -Syy`刷新软件源完成工作。

####更新系统并进行linux基本设置
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

# 在root ALL=(ALL) ALL的下一行加入以下內容:

user ALL=(ALL) ALL
```
最后就可以切换到普通用户账号了，之后都是用这个普通用户账号进行操作。

```
su user
```

###安装所需工具与桌面环境
1.安装中文字体，SSH与一些工具

```
sudo pacman -S networkmanager xorg xorg-server pulseaudio noto-fonts git openssh fakeroot base-devel
```

2.（可选）安装AUR助手yay
AUR是arch linux的用户软件源，可以从里面搞到很多有用的软件，不过我们运行莉莉丝的王座不需要用到它，因此是可选的。

```
sudo pacman -S --needed git base-devel && git clone https://aur.archlinux.org/yay.git && cd yay && makepkg -si
```

3.安装桌面环境
