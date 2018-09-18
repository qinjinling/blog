## SDK 及其周边

首先你得安装 [SDK Tools](http://www.androiddevtools.cn/#sdk-tools)，利用 SDK Manager 去安装安卓开发的其他工具。至于要安装哪些工具，这里有一个官方的指导原则：
- SDK Tools 必须
- SDK Platform-tools 必须
- SDK Platform 必须至少安装一个版本，建议安装最新版本，原因看下面
- System Image 建议安装
- Android Support 建议安装
- SDK Samples 建议安装

对于初学者，很可能会困惑于到底安装哪个版本。

> 永远只用最新的 SDK 版本,这是 Google 官方强烈建议的。你的 app 能运行的 Android 版本不是由 SDK 决定的，是由每一个项目的 minSDK 决定的。SDK 都是向下兼容的。SDK 在不断改进中，新的 SDK 会提供更强大开发工具，而且用 4.0 的 SDK 编译的 2.1 的 apk 的执行效率会比用 2.1 的 SDK 编译的更高。    至于每个 app 应该用什么 minSDK ，应该根据应用具体的 API 来，如果 app 没有用到 1.6 以上 SDK 新提供的 API，那么用 1.6 会在提供相同体验下反而会比 2.1 兼容更多机型.  那些扯 2.x 的就别误人子弟了。—— [Nerd Leo 知乎的回答](https://www.zhihu.com/question/19978657/answer/13551186)

按照 [AndroidDevTools](http://www.androiddevtools.cn/) 配置镜像服务器。
## Android Studio 安装

下载最新的 [Android Studio](http://www.androiddevtools.cn/#android-studio) 版本，免安装的，下载后解压缩到指定目录即可。
## Gradle 配置

Android Studio 安装之后，启动，会自动下载 Gradle，好在 Gradle 官网没有被墙，但下载速度也是慢的出奇，耐心等待即可。如果你按照上面的步骤完成之后，打开 Settings --> Build --> Build Tools --> Gradle，会发现有两个和 Gradle 有关的目录，配置成你的目录即可。

如果你工作的网络环境是使用代理的，新建一个文件 gradle.properties 分别放在这两个目录下面，文件内容如下：

```
# http
systemProp.http.proxyHost=你的代理服务器
systemProp.http.proxyPort=代理服务器端口
systemProp.http.proxyUser=userid
systemProp.http.proxyPassword=password
systemProp.http.nonProxyHosts=
# https
systemProp.https.proxyHost=你的代理服务器
systemProp.https.proxyPort=代理服务器端口
systemProp.https.proxyUser=userid
systemProp.https.proxyPassword=password
systemProp.https.nonProxyHosts=
```
## Android Studio 简单设置
### 界面设置

默认的 Android Studio 为灰色界面，可以选择使用炫酷的黑色界面。Settings --> Appearance --> Theme，选择 Darcula 主题即可。
### 编程字体设置

此部分会修改编辑器的字体，包含所有的文件显示的字体。Settings --> Editor --> Colors & Fonts --> Font。默认系统显示的 Scheme 为 Defualt，你是不能编辑的，你需要点击右侧的 Save As...，保存一份自己的设置，并在当中设置。之后，在 Editor Font 中即可设置字体。Show only monospaced fonts 表示只显示等宽字体，一般来说，编程等宽字体使用较多，且效果较好。
### 默认文件编码

无论是你个人开发，还是在项目组中团队开发，都需要统一你的文件编码。出于字符兼容的问题，建议使用 utf-8 。中国的 Windows 电脑，默认的字符编码为 GBK。Settings --> File Encodings，建议将 IDE Encoding、 Project Encoding、 Properties Fiels 都设置成统一的编码。
### 快捷键

Android Studio 的快捷键和 Eclipse 的不相同，但是你可以在 Android Studio 中使用 Eclipse 的快捷键。
Settings --> Keymap。你可以从 Keymaps 中选择对应IDE的快捷键，Android Studio 对其他 IDE 的快捷键支持还是比较多的。建议不使用其他 IDE 的快捷键，而是使用 Android Studio 的快捷键。

当你想设置在某一个快捷键配置上进行更改，你需要点击 copy 创建一个自己的快捷键，并在上面进行设置。Android Studio 默认的快捷键中，代码提示为 Ctrl+Space，会与系统输入法快捷键冲突，需要特殊设置。Main menu --> Code --> Completion --> Basic，更改为你想替换的快捷键组合。
### 其他设置
- 显示行号 Settings --> Editor --> Appearance，勾选 Show line numbers
- 去除拼接检查。我个人觉得没用，所以禁用掉。Settings --> Inspections --> Spelling，取消勾选
- 插件。Android Studio 和 Eclipse 一样，都是支持插件的。Android Studio 默认自带了一些插件，如果你不使用某些插件，你可以禁用它。Settings --> Plugins，右侧会显示出已经安装的插件列表。取消勾选即可禁用插件。
  个人禁用的一些插件：
  - CVS Integration：CVS 版本控制系统，用不到。
  - hg4idea：Mercurial 版本控制系统，用不到。
## 安装 Genymotion 模拟器

Genymotion 安卓模拟器的优点就是快，当然，如果你能真机调试的话，最好用真机调试吧。

访问 [Genymotion 官网](https://www.genymotion.com/download/)，下载安装程序，然后系统会让你先注册一个账号。成功注册后直接下载 with VirtualBox 版本。安装成功后，直接打开主程序，在 Settings --> Account 选项卡里面输入账号密码登录。之后在程序主界面点击 Add 按钮，选择一个虚拟设备进行安装。注意：如果你是刚刚注册，选择虚拟设备的时候列表有可能是空的，稍候再试即可。下载虚拟设备的时间可能有点长，请耐心等待。
### Unable to load R3 module 解决方案

如果以上步骤都成功了，而你用的是 Windows 7 64 位的话，在 Genymotion 里面启动虚拟设备的时候可能会遇到错误，提示无法启动，让你直接打开 Oracle VM VirtualBox 启动，VirtualBox 启动后报出 Unable to load R3 module 的错误，直接去 Google 一下解决，这里直接给出解决方案。
1. 重命名 C:\Windows\System32\uxtheme.dll 文件为 uxtheme_bak.dll
2. 下载 win7 x64 原版文件 uxtheme.dll，拷贝到 C:\Windows\System32\ 目录

之后重启 Genymotion，启动虚拟设备即可。
### INSTALL_FAILED_NO_MATCHING_ABIS 解决方案

是由于使用了 native libraries。该 native libraries 不支持当前的 CPU 的体系结构。解决办法：将你的虚拟器运行起来，下载 genymotion-arm-translation_v1.1.zip，用鼠标将此压缩包拖到虚拟机窗口中，出现确认对跨框点 OK 就行。然后重启你的虚拟机。
