---
title: Win10系统下的Windows Terminal+WSL配置指南
date: 2020-08-15 23:38:53
tags: 
- WSL
- Windows Terminal
- terminal
categories: 技术
---

在计算机学习的过程中，我们常常会纠结于 Windows、Linux 和 Mac OS 的选择。Mac OS X 凭借其出色的GUI与天然的 Unix-like 环境，倍受程序猿欢迎。但对于大部分同学来说，Macbook 系列的价格较为高昂，因而也自然无法接触到 Mac OS X 了。那么开发环境的选择一般就只剩下纯 Windows、Windows+虚拟机 和 Windows+双系统了。纯Windows环境往往会增加折腾过程的难度，同时Windows 老旧的命令行界面也让人不那么提得起劲。（~~毕竟好看才是第一生产力~~）后两种固然能够体验最纯粹的 Gnu/Linux 环境，但虚拟机的启动等待和系统的切换等待让这一过程显得不那么顺滑。

那么，有没有一种方法能让我们能够顺滑高效地体验Linux的魅力呢？

有！那就是上手微软的 **Windows Terminal** 和 **Windows Subsystem for Linux**（WSL）了！
<!--more-->
***

## Why Windows Terminal and WSL？

微软的 Windows Terminal 发布于2019年，它是一个现代的终端工具，集成了 CMD、PowerShell、WSL 这三大环境，同时还内置了SSH 客户端，让用户能够在同一个窗口中**顺滑**地使用不同的环境与工具。

而 WSL 则是内嵌于 Windows 的 Linux 子系统，它允许开发人员直接在Windows系统中"套娃"运行一个 GNU/Linux 环境，在其中你可以使用各类的命令行工具和应用程序，避免使用虚拟机或双系统设置的庞大开销。WSL 2 则是 WSL 的升级版本，它提供了完整的 Linux 内核，提升了文件 IO 性能，并支持 Docker 的使用，不过这也导致了其跨 OS 文件系统的性能不如 WSL 1， 即在访问 Windows 的文件系统时会比 WSL 1慢一点。

个人认为，如果想体验较为纯正的 Linux 环境，并且希望在其中获得更好的 IO 性能，推荐使用 WSL2。但若有跨 OS 使用文件的需求，则可以考虑 WSL 1。如果暂时还没有考虑好自己的需求的话，任意选择一个先折腾起来也不要紧，因为WSL可以很方便地在1、2版本间切换（后面会提到）。

下面贴一张微软官网上的 WSL 1和 WSL 2的性能对比图：

![比较WSL 1和WSL 2](https://charfole-blog.oss-cn-shenzhen.aliyuncs.com/image/53aef1cc2413b3fe50469c5e3a38723.png)


***
## Windows Terminal安装与初步体验

在安装之前，请先 Win+R 并键入winver来确认自己 Win10 版本，**Windows Terminal的最低要求为 Win 10-18362 及以上的版本**，而后续 **WSL 2 的安装则至少需要Win 10-2004-19041版本**，如果版本过低，请先进行系统更新。如系统无法自动升级，可以通过[Microsoft的升级助手](https://www.microsoft.com/zh-cn/software-download/windows10)升级版本。在这里，我建议最好确保电脑升级到了 Win10-2004-19041版本，否则无法体验 WSL 2（笔者成功更新到2004版本之前系统还是一年前的1903版本，更新后一切正常，所以大家可以放心~）

折腾完更新后，我们马上来安装Windows Terminal，安装过程十分简单，只需要在电脑上打开[Microsoft Store](https://aka.ms/terminal)（微软商店），搜索Windows Terminal并安装即可。安装完启动，便可以愉快地玩耍啦！

![Terminal](https://charfole-blog.oss-cn-shenzhen.aliyuncs.com/image/1342f00573e07513d603a8fa7dc2251.png)

从图中我们可以看到，在 Windows Terminal 中通过选项卡我们可以随意切换 Powershell、CMD、WSL 等多个环境。使用快捷键Ctrl+Shift+1/2/3/4...可以快速地打开不同的窗口，使用 Alt+shift+减号（加号）可以实现水平（垂直）分屏。通过 settings.json 文件，我们还可以让 Terminal 变得更酷，在本文的第四点将进行详细的描述。

***

## WSL-Ubuntu 18.04安装与体验

安装完 Windows Terminal 后，下一步我们来安装 WSL 。正常情况下，微软商店会默认将WSL下载到系统驱动器中（也就是大部分人容量较少的C盘），如果你目前的C盘空间不是很足（少于50G），那么我建议你按照下面的步骤来将WSL安装在其他位置。

### 启用系统中的WSL和虚拟机平台功能

首先以管理员模式启动 Powershell，其后分别输入以下代码：

```powershell
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

上文的第一串代码是开启系统的 WSL 功能，第二串则是开启 ”虚拟机平台“ 可选组件，重启电脑后两项功能便已打开。

### 安装Linux分发版到电脑的任意位置

（写在前面：如果你觉得将WSL安装在系统盘问题也不大，那么可以直接在微软应用商店中安装你想要的WSL版本，完成密码设置后即可跳到下一步）

由于我们需要跳开默认的安装位置，因此我们需要手动安装一下 Linux 分发版。首先，我们从[微软的文档](https://docs.microsoft.com/zh-cn/windows/wsl/install-manual)中找到可供手动下载的WSL目录，选择自己需要的版本（Ubuntu、Debian、Kali等），下载其对应的.appx文件。第二步，将下载好的以.appx后缀结尾的文件更名为.zip文件，之后解压到你自定义的位置（系统盘或非系统盘）。最后，点击解压后出现以分发版本命名的.exe文件（比如，下载的是Ubuntu-18.04版本，则对应文件名为 "ubuntu1804.exe"），等待安装完成后，输入你的账户和密码，便完成了 WSL 的初步安装！

![Ubuntu](https://charfole-blog.oss-cn-shenzhen.aliyuncs.com/image/image-20200813162244438.png)

（PS: 如果你在设定或输入密码时屏幕没有显示，请不要担心，这是正常现象，想进一步了解该密码的作用可戳[这里](https://docs.microsoft.com/zh-cn/windows/wsl/user-support)）



### 在Windows Terminal中管理与体验WSL

Win+R+输入wt快速打开Windows Terminal后，在Powershell中通过以下的命令来管理我们的WSL。

```powershell
# 检查分配给每个已安装的 Linux 分发版的 WSL 版本
wsl --list --verbose # 等同于wsl -l -v
# 将某个Linux分发版设置为受某一 WSL 版本
wsl --set-version <Linux 分发版的名称> <版本号：1、2>
# 将WSL 2设置为默认Linux分发版的默认版本
wsl --set-default-version 2
# 再次提醒，以上三条命令中的后两条需要用到WSL 2，而WSL 2仅支持Win10 2004以上的版本（前文第二点已提到）
```

完成以上的设定后，点击选项卡中的下拉框，选择安装好的WSL版本便可以开始愉快的玩耍啦~可以看到，原来Windows中的文件均被挂载到了/mnt 目录下（C盘在 /mnt/c，D盘在 /mnt/d，依此类推），我们可以很方便地进行跨文件系统的访问。

![image-20200814230639846](https://charfole-blog.oss-cn-shenzhen.aliyuncs.com/image/image-20200814230639846.png)

到了这里，Windows Terminal和WSL的基本配置就已经完成了！如果你想进一步对Windows Terminal和WSL进行美化（如上图），那么强烈建议你继续阅读下一部分内容。
***
## WSL美化与Windows Terminal美化

### 安装 oh-my-zsh

为了让WSL的shell界面变得更加美观，笔者推荐安装 [oh-my-zsh](https://github.com/ohmyzsh/ohmyzsh)，它是一款针对zsh终端的扩展框架，支持配置多种插件与主题来丰富我们的终端。

首先，我们需要安装zsh来作为我们的 shell，打开 WSL，键入以下代码：

```shell
sudo apt-get install zsh # Ubuntu下安装zsh，安装后重启wsl
echo $SHELL # 查看当前使用的shell是否为zsh
# 如果显示为/usr/bin/zsh，则代表安装成功，否则使用以下命令切换为zsh
chsh -s /bin/zsh # 将shell切换为zsh
```

之后，我们通过curl来安装下载oh-my-zsh。(如果遇到了无法下载Github文件的问题，可以参考一下[这一篇文章](https://shileiye.com/835)来修改host文件)

```shell
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

安装的过程中需要获取一些权限，输入y（yes）就好。安装完成后，等待WSL自动重启，此时便用上了oh-my-zsh。

### 配置使用 oh-my-zsh

上一步的折腾成功后，我们就离WSL美化大功告成只差一步了。首先，进入WSL，运行以下命令：

```shell
# 利用vim编辑器打开zsh配置文件
vim ~/.zshrc

# 修改oh-my-zsh的主题，这里推荐使用agnoster主题（也可选择其他）
ZSH_THEME="agnoster"

# 保存文件后退出回到Shell，然后输入
source ~/.zshrc
```

以上的操作要求对Vim的使用有所了解，如果你第一次接触Vim，不妨先快速浏览一下这篇[文章](https://www.jianshu.com/p/8b679b35c9d5)。

修改完主题后，因为缺少一个合适的字体，所以主题的图标显示有所异常。因此，我们需要安装一下[powerline](https://github.com/powerline/fonts)字体到我们的**Windows系统**中（非WSL）来解决这个问题。

我们可以根据 Github 的指引，在 Powershell 中使用git来安装。

```shell
# 下载
git clone https://github.com/powerline/fonts.git --depth=1
# install
cd fonts # git clone的文件夹
./install.sh
```

如果你不太熟悉git安装，也可以下载仓库的zip文件，解压后双击运行文件夹中的 “install.sh”文件或者“install.ps1”文件即可。

安装完字体后，我们再次进入Windows Terminal，在下拉框中选择设置。

![设置](https://charfole-blog.oss-cn-shenzhen.aliyuncs.com/image/image-20200813234409331.png)



进入设置文件 “settings.json” 后，找到 “profiles” 字段下的 “list” 字段，在WSL配置模块中增加或修改 ```fontFace``` 字段为你想要的字体名称。（字体名称改为刚刚安装好的 Powerline 字体中的一种，我使用的是DejaVu Sans Mono for Powerline字体）

![设置](https://charfole-blog.oss-cn-shenzhen.aliyuncs.com/image/image-20200813235847851.png)



VS Code中使用WSL也会出现乱码情况，解决的方法类似，也是下载字体与修改配置。因为篇幅有限，关于VS Code下的 WSL 配置就不展开叙述了，详情可参考这两个链接：[适用于WSL的VS Code入门](https://docs.microsoft.com/zh-cn/windows/wsl/tutorials/wsl-vscode)和[解决VS Code中的WSL乱码](https://zhuanlan.zhihu.com/p/68336685)。

如果想要WSL更酷炫一点，可以进行[neofetch](https://github.com/dylanaraps/neofetch)的配置，Ubuntu中的操作如下：

```shell
# Ubuntu下安装neofetch
sudo apt install neofetch

# 利用vim编辑器打开zsh配置文件
vim ~/.zshrc

# 在配置文件中主题行的后一行添加neofetch（在文件的末尾加也可以）
ZSH_THEME="agnoster"
neofetch
```

### 安装zsh常用插件

zsh中有许多优秀的插件帮助我们提高效率，下面介绍两个常用插件的安装流程，分别是语法高亮插件和自动提示插件。首先，将当前路径切换到 plugins 目录中。

```shell
cd ~/.oh-my-zsh/custom/plugins
```

之后，我们用git下载这两款插件到 WSL 中。

```shell
# git下载 zsh-syntax-highlighting
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

# git下载 zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```

下载完成后，我们修改一下zsh的配置文件，使插件生效。

```shell
# 首先用vim进入.zshrc配置文件
vim ~/.zshrc

# 之后利用vim编辑文件为
plugins=(
        zsh-syntax-highlighting
        zsh-autosuggestions
        )
source $ZSH/oh-my-zsh.sh
source $ZSH_CUSTOM/plugins/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh
source $ZSH_CUSTOM/plugins/zsh-autosuggestions/zsh-autosuggestions.zsh
```



配置好oh-my-zsh并安装好插件后，WSL方面的美化工作就基本完成啦！！最终，我们的WSL变成下图所示的样子。（~~真香！~~）

![WSL Final](https://charfole-blog.oss-cn-shenzhen.aliyuncs.com/image/image-20200814104151498.png)



### Windows Terminal 美化

​		Windows Terminal的配置文件是一个 JSON 格式的文件，它支持我们对Terminal进行大规模地自定义，关于该 JSON 的一切信息都可以在[微软文档](https://docs.microsoft.com/zh-cn/windows/terminal/customize-settings/global-settings)中找到。下面我们来看看设置文件中最主要的部分，并简要介绍前三部分。

- 全局设置：位于 JSON 文件的最顶端，主要用于设置Windows Terminal的亮暗主题、和启动时的默认环境

- 环境入口 ```profiles```：其中包含defaults和list两个字段，defaults中的设置默认对所有的环境生效，list中可以对每个环境进行细化的定义，这些定义会覆盖defaluts中的内容。

- 配色方案 ```schemes```：这里可以自定义多个主题配色，在profiles中可以为不同的环境分配不同的配色。

- 键绑定 ```keybindings```：自定义多种快捷键。

  ​	

  **全局设置**中，都是一些Terminal里的宏观设置，比较重要的有这两项：

- 亮暗主题设置：```"theme":"system"(default)、"dark"、"light"```

- 初始配置：```"defaultProfile":"{默认为Powershell的GUID}"```，其中 GUID 为不同环境的唯一标识码，可以在 profiles 中的 list 找到你想默认启动的环境，将其GUID作为defaultProfile的值。

  

  在环境入口**profiles**中，各个环境通用的设置可以放在defaults字段中，而每个环境特定的配置可以在 list 中准确定义。（字体、背景、配色方案等，详情可看[这里](https://docs.microsoft.com/zh-cn/windows/terminal/customize-settings/profile-settings)）

​		我在Profiles中使用的配置项有这些，其中的照片、图标、字体、颜色主题等可以自行更换。

```json
"useAcrylic": true, // 推荐启用毛玻璃
"acrylicOpacity": 0.6, // 启用毛玻璃后，可调整背景透明度
"icon": "ms-appx:///ProfileIcons/{9acb9455-ca41-5af7-950f-6bca1bc9722f}.png", //图标
"backgroundImage": "C:\\Users\\13359\\Pictures\\Camera Roll\\wolf.png", //背景图片
"backgroundImageOpacity": 0.3, //图片透明度
"backgroundImageStretchMode": "fill", //填充模式
"fontFace": "DejaVu Sans Mono for Powerline", //字体种类
"suppressApplicationTitle": true,//固定选项卡中的标题
"fontSize": 12, //文字大小
"colorScheme": "cyberpunk", //主题，与schemes中的名称对应
"cursorColor": "#FFFFFF", //光标颜色
"cursorShape": "vintage", //光标形状
"startingDirectory":"C://" //起始目录
```



在配色方案schemes中，我们可以定义多个不同的颜色主题，并在不同的环境中运用不同的颜色主题，以获取最适合自己的终端体验。以下代码是两种配色方案的定义，其中 ```name```用于命名配色方案，并用于赋值给环境设置中的```colorScheme```字段。关于更多配色方案的选取可以参照Iterm2-color-schemes的[网站](https://iterm2colorschemes.com/)和[Github](https://github.com/mbadolato/iTerm2-Color-Schemes)。

```json
"schemes": [
        {
            "name": "cyberpunk",
            "black": "#000000",
            "red": "#ff7092",
            "green": "#00fbac",
            "yellow": "#fffa6a",
            "blue": "#00bfff",
            "purple": "#df95ff",
            "cyan": "#86cbfe",
            "white": "#ffffff",
            "brightBlack": "#647da1",
            "brightRed": "#ff8aa4",
            "brightGreen": "#21f6bc",
            "brightYellow": "#fff787",
            "brightBlue": "#1bccfd",
            "brightPurple": "#e6aefe",
            "brightCyan": "#99d6fc",
            "brightWhite": "#ffffff",
            "background": "#332a57",
            "foreground": "#e5e5e5"
        },
        {
            "name": "DimmedMonokai",
            "black": "#3a3d43",
            "red": "#be3f48",
            "green": "#879a3b",
            "yellow": "#c5a635",
            "blue": "#4f76a1",
            "purple": "#855c8d",
            "cyan": "#578fa4",
            "white": "#b9bcba",
            "brightBlack": "#368bfa",
            "brightRed": "#eb001f",
            "brightGreen": "#0f722f",
            "brightYellow": "#eef107",
            "brightBlue": "#428ffa",
            "brightPurple": "#fb0067",
            "brightCyan": "#2e706d",
            "brightWhite": "#fdffb9",
            "background": "#1f1f1f",
            "foreground": "#b9bcba"
        }
    ],
```



我将上面的两种配色方案分别应用在 Powershell 和 WSL 中，这个操作十分简单，只需要在 profiles 的 list 中将不同颜色主题的名字分别赋值到 Powershell 和 WSL 中的 ```colorScheme```中即可。

![image-20200814231326728](https://charfole-blog.oss-cn-shenzhen.aliyuncs.com/image/image-20200814231326728.png)



到了这里，我们 Windows Terminal 和 WSL 的美化工作就**大功告成**啦！！最终的成果请参看下图~

![Powershell](https://charfole-blog.oss-cn-shenzhen.aliyuncs.com/image/image-20200814161425692.png)

![WSL](https://charfole-blog.oss-cn-shenzhen.aliyuncs.com/image/image-20200814161521972.png)

***

## 结语与相关链接

感谢你看到这里，这篇文章是我断断续续构（mo）思（yu）了两三天才写好的，如果它对你有一点小小的帮助，那就是我最大的欣慰了。同时，这也是我第一次写这么详细的技术分享文章，因此难免会有一些纰漏出现。如果你发现了有疑问或者有错误的地方，不妨在在下方讨论或指出，**再次感谢！**

在撰写文章时，我从很多地方获得了帮助与指引，如果你想了解更多关于 Windows Terminal 和 WSL 的内容，可以浏览以下链接：

- [微软 Windows Terminal 使用文档](https://docs.microsoft.com/zh-cn/windows/terminal/)
- [微软 WSL 使用文档](https://docs.microsoft.com/zh-cn/windows/wsl/)
- [少数派：Windows Terminal 自定义教程](https://sspai.com/post/59380)
- [知乎：WSL + oh-my-zsh 配置教程](https://zhuanlan.zhihu.com/p/68336685)
- [掘金：WSL 美化](https://juejin.im/entry/6844903847794573319)
- [菜鸟教程：vim 的使用](https://www.runoob.com/linux/linux-vim.html)