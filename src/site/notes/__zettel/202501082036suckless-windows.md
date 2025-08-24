---
{"dg-publish":true,"permalink":"/__zettel/202501082036suckless-windows/","title":202501082036,"tags":["定制","windows","suckless","msys2"],"created":"2025-01-08T20:36:52+08:00"}
---


Windows毛病例举，

- 祖传蓝屏（这一点在xp尤为严重，win7以上已经少见许多）。
- 静默更新。悄悄下载更新，强制更新，莫名其妙跳出来说要更新，关机时强迫更新。恶心的默认策略！
- 万年难用的资源管理器，不带标签页，窗口开一堆。
- 开发上来说，编译工具链体验比不上linux.

虽然Windows很难用，但并不妨碍它桌面市场占用率第一的地位。在生活或者工作中，我还是无法避免地被动使用windows. 因此，我希望通过一些简单定制，让我的使用体验能好一些。

安装MSYS2
---

如果你在windows上做开发，必定会装git bash，而它提供的终端就是MINGW（Minimalist GNU for Windows）。它的主要作用是提供windows版本的gcc编译工具链。它自带了一些linux基础命令，仅仅是基础。所以[MSYS2][1]出现了，它在MINGW的基础上，提供了更多的命令，并且自带了一个著名的包管理器—[Pacman][2]. 没错就是Archlinux的包管理器。如此一来便可以在在windows上使用相同的linux命令行，而这些都是适配原生windows的，并非虚拟机或者子系统之类的。并且想要啥软件包，直接通过pacman安装也非常方便。

### MinGW、Clang、UCRT？

安装完之后，会产生多个launcher，选择哪个呢？[这里][3]有详细的对比。其实就是工具链和运行时库的区别。一般用gcc建议选择MINGW64，用llvm建议使用CLANG64.

### 一点配置

网上的配置[帖子][4]还是比较多的，这里简单列举几个常用的。

#### 修改镜像

`/etc/pacman.d` 文件夹里面包含几个mirrorlist，新版的MSYS2里面默认已经有中科大源。直接编辑将其提前即可。

```
Server = http://mirrors.ustc.edu.cn/msys2/
Server = https://mirrors.tuna.tsinghua.edu.cn/msys2/
```

#### 禁用某些源

编辑/etc/pacman.conf，注释掉不想要的源。

> [!attention]
> vim和zsh在mingw里面，mingw64和clang64里面都没有。

参考：https://my.oschina.net/zuozhihua/blog/8757266

#### 继承环境变量

如果你之前安装过git bash，现在你的电脑里面应该有两套mingw（一个git bash安装的，一个是MSYS2安装的）。Git bash里面自带vim. 而MSYS2里面可以通过包管理器安装vim. 然而重要的是windows系统程序的路径需要继承下来，这样可以方便的在mingw里面直接调用windows程序例如

- notepad.exe：记事本
- ：命令提示符
- explorer：文件管理器

要继承windows系统环境变量，找到MSYS2的安装目录（例如，D:\msys64）下有mingw64.ini，将里面的

```
MSYS2_PATH_TYPE=inherit
```

取消注释，再次打开mingw64即可继承系统环境变量。

> 上面只改了MinGW64的launcher，如要改其他可如法炮制。

#### 更改默认shell

launcher.ini（例如mingw64.ini）里面添加环境变量

```
SHELL=/usr/bin/zsh
```

参考：https://superuser.com/questions/961699/change-default-shell-on-msys2

#### 添加到vscdoe集成终端

设置里面加上，参考：https://gist.github.com/dhkatz/106891324c9624074a84d11e2691144b

```json
{
    "terminal.integrated.defaultProfile.windows": "MSYS2 MINGW64",
    "terminal.integrated.profiles.windows":{
        "MSYS2 MINGW64": {
            "path": "D:\\msys64\\usr\\bin\\bash.exe",
            "args": ["--login", "-i"],
            "env": {
                "MSYSTEM": "MINGW64",
                "CHERE_INVOKING": "1",
                "MSYS2_PATH_TYPE": "inherit"
            }
        },
        "PowerShell": {
            "source": "PowerShell",
            "icon": "terminal-powershell"
        },
        "Command Prompt": {
            "path": [
                "${env:windir}\\Sysnative\\.exe",
                "${env:windir}\\System32\\.exe"
            ],
            "args": [],
            "icon": "terminal-"
        },
        "Git Bash": {
            "source": "Git Bash"
        }
    }
}
```

安装Everything
---

Everything是一个windows文件搜索程序，搜索速度快，轻量易用，支持正则表达式。完爆windows资源管理器自带的搜索功能。直接至官网下载安装即可。

安装QtTabBar
---

[QtTabBar][5]是一款windows资源管理器扩展工具。之前说了，windows自带的文件管理器不支持标签页。这才有了QtTabBar. 值得一提的功能点：

- 深度资源管理器标签支持，甚至控制面板都是一个标签页的事儿。
- 支持用户自定义组，放置常用路径（类似于书签）。
- 支持多语言，官网下载翻译包即可。
- 桌面快捷操作。桌面双击即可触发浮动菜单，常用路径一键即达。
- 不俗的文件预览功能。鼠标悬停即可预览目录、图片、视屏等。预览图片真的很实用！

微软拼音添加小鹤双拼
---

作为双拼爱好者，当初去知乎吸收了一下各种双拼方案对比，决定学小鹤，至今已经忘记它有啥优缺点了，反正用着挺好就是了。

方案来自互联网。在CMD中直接粘贴

```
reg add HKCU\Software\Microsoft\InputMethod\Settings\CHS /v UserDefinedDoublePinyinScheme0 /t REG_SZ /d "小鹤双拼*2*^*iuvdjhcwfg^xmlnpbksqszxkrltvyovt" /f
```

PS，微软大法好！在曾经的XP时代，搜狗拼音还是很好用的，如今已是臃肿不堪。后来一直在找替代，知道微软拼音横空出世。至此windows输入法无需再纠结。其界面节约，打字流畅，又支持添加小鹤双拼，已经很够用了！

另辟蹊径之linux虚拟机
---

无论是WSL还是MSYS2，启动起来都有些卡，而且换zsh之后就更卡了。目前还不清楚原因，所以才有了这种尝试。Windows下可以使用virtualbox安装一个linux虚拟机，然后开启sshd，配置好端口转发（或者虚拟网卡）即可在宿主机上ssh连接虚拟机，体验上和linux远程开发机相差无几。

使用VirtualBox安装一个Archlinux虚拟机重点步骤：

1. 安装VirtualBox
2. 下载archlinux镜像（LiveCD）
3. 从镜像启动虚拟机，执行安装流程，这一步参考archwiki中的installation guide

值得注意的是，虚拟磁盘创建分区的时候，建议直接选择[MBR分区表][6]。我一开始选择GPT分区表，多挂载一个boot分区，反复装了3次，装完启动引导重启都提示找不到可启动的设备。反正是虚拟机，也不用考虑引导多个系统，直接MBR分区表，还可以少挂载一个boot分区。

装完之后安装一些常用软件，

- vim
- zsh
- openssh
- **NetworkManager**

> [!attention] 
> LiveCD里面网络配置是开箱即用的，但这并不表示装好的系统网络配置和LiveCD一样。当你兴奋地重启进新系统之后，很可能发现里面没网。因此，一定要在安装过程中（arch-chroot）把[网络管理器][7]的包装好。

启动进新系统之后，

```bash
systemctl start NetworkManager.service  # 启动网络管理器
systemctl enable NetworkManager.service # 设为自动启动
```

### 配置端口转发

系统启动之后，不必装图形界面。直接启动一个ssh sever，然后在windows上ssh连上即可。使用体验和远程Linux开发机一致。但宿主机（Host）和虚拟机（VM）的网络不一定直接相通，还须做一点配置。

端口转发配置参考[SSH连接NAT网络模式VirtualBox虚拟机][8]。

![](https://oscimg.oschina.net/oscnet/74b6fb8a7d1a0b05b644a38bba037cdfd8e.jpg)

### 配置固定IP

端口转发配置简单易用，但有一个小缺点。每次暴露一个端口，都需要新增一条规则。比如我转发22端口给ssh，那我sftp的23端口呢？这时候又得新增一条规则，比较麻烦。

我们可以通过增加一张虚拟网卡，配置静态IP地址和主机网络互通。此后直接访问这个IP带端口即可访问虚拟机。配置参考[Win10宿主机ssh连接VirtualBox里的archlinux虚拟机][9]。

需要注意一点，配置接口（interface）静态地址并使之持久化生效是网络管理器的职责。所以一定要通过网络管理器的接口设置IP地址。之前安装的NetworkManager提供`nmcli`命令行配置工具。详细用法参考[NetworkManager - ArchWiki][10].

使用示例，

```bash
yychi@~/bin> nmcli dev
DEVICE          TYPE      STATE         CONNECTION 
wlp2s0          wifi      已连接        LovelyLife 
lo              loopback  连接（外部）  lo

yychi@~/bin> nmcli connection
NAME                  UUID                                  TYPE      DEVICE 
LovelyLife            92506e38-1823-4f66-1a89-75a62ecabe74  wifi      wlp2s0 
lo                    7e2fe31d-8ba1-1823-ad29-239857601abb  loopback  lo     

yychi@~/bin> nmcli connection edit LovelyLife

===| nmcli 交互式连接编辑器 |===

正在编辑已有的连接 "802-11-wireless"："LovelyLife"

输入 "help" 或 "?" 查看可用的命令。
输入 "print" 来显示所有的连接属性。
输入 "describe [<设置>.<属性>]" 来获得详细的属性描述。

您可编辑下列设置：connection, 802-11-wireless (wifi), 802-11-wireless-security (wifi-sec), 802-1x, ethtool, match, ipv4, ipv6, hostname, link, tc, proxy
nmcli> set ipv4.addresses 192.168.56.100/24
nmcli> set ipv4.gateway 192.168.56.1
nmcli> save 
persistent  temporary   
nmcli> save persistent 
成功地更新了连接 "MeijmTenda" (ab876b5a-a02e-43c5-8d65-8da29891b0c1)。
```

### 配置ssh别名

在宿主机（Host）上配置，

```ini
# FILE:~/.ssh/config

Host varch
    HostName 192.168.56.100
    User xxxx
```

此后就可以直接`ssh varch`了。

小结
---

至此，我在windows上安装了一些小工具、配置了一个可用的linux环境。与我个人来说，大幅提升了windows的易用性。


[1]: https://www.msys2.org/
[2]: https://wiki.archlinux.org/title/Pacman
[3]: https://www.msys2.org/docs/environments/
[4]: https://www.cnblogs.com/wswind/p/10650126.html
[5]: http://qttabbar.wikidot.com/
[6]: https://wiki.archlinux.org/title/Partitioning#Example_layouts
[7]: https://wiki.archlinux.org/title/Network_configuration#Network_managers
[8]: https://www.cnblogs.com/phoebus-ma/p/18219807
[9]: https://www.cnblogs.com/arisskz6/p/16983492.html
[10]: https://wiki.archlinux.org/title/NetworkManager#Edit_a_connection