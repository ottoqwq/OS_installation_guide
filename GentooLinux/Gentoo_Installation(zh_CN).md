# Gentoo Linux

<img src="mudrvB7q.png" style="zoom:67%;" />

*前排提示：实际安装时可以使用archiso来构建，其自带的genfstab,zsh高亮等也许会对你有用（安装时一样把根目录挂在/mnt/gentoo下）；也可以在livecd上使用systemd-nspawn*

本篇文章主要介绍如何在Gentoo Linux上安装一个系统，并且安装一些必要的软件，以及如何使用Gentoo Linux的一些特性。相信，只要你安装了Gentoo Linux，你就会觉得自己的系统更加美好了。（如果你不知道Gentoo Linux，可以先看看[Gentoo Linux官网](http://www.gentoo.org/)）

>为什么这个 Linux 发行版值得大家作为自己的生产力操作系统呢？
`灵活性`，Gentoo Linux 通过一个叫做 USE 的功能实现对系统最优化配置，安装系统的人员（下面我叫系统管理员），要对自己需要什么，系统用来做什么，系统的构成，做到心中有数。而 USE 的意义就在于此，能够最大化帮助系统管理员管理系统，优化系统；
`高效性`，因为 Gentoo Linux 的安装决定了这个系统本身甚至比 Archlinux 更加 KISS（Keep It Simple And Stupid），因 USE 能够为系统减负，构建更小、更高效的系统；
`时刻保持最新`，这个观点因用户选择的更新通道不同而区别，在 ~amd 这个更新通道，大部分软件能够时刻保持最新，如果是 amd64 稳定分支，有时候软件不一定是最新版本。其余的情况就看 Gentoo 开发社区是否有人手能够维护。
——Hougelangley

本篇不会介绍如何配置内核（配置因机子而异，有兴趣的可以参考阿洲先生的文章）。如果本篇有不足之处或是您有更好的建议，欢迎留言或者联系Otto。

## 联网

先用`ifconfig`来确定网卡设备名(一般是wlan0,ra0，wlp9s0什么的)

`net-setup wlan0`

ping来确定是否连上了。

```bash
ping gnu.org
PING gnu.org (114.514.191.181) 56(84) 字节的数据包，平均时间=1ms TTL=64
```

若无法ping到,详情可以看Gentoo wiki里面[配置网络](https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Networking/zh-cn#.E4.BD.BF.E7.94.A8DHCP)的部分。

## 准备磁盘

磁盘分区方面可以有很多选择，下面仅仅列出默认分区方案

```bash
fdisk /dev/sda #分区工具有很多，个人习惯还是喜欢fdisk
```


```bash
#下文作为一个分区示范
Command (m for help):p
Disk /dev/sda: 28.89 GiB, 31001149440 bytes, 60549120 sectors
Disk model: DataTraveler 2.0
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 21AAD8CF-DB67-0F43-9374-416C7A4E31EA
 
Device        Start      End  Sectors  Size Type
/dev/sda1      2048   526335   524288  256M EFI System
/dev/sda2    526336  2623487  2097152    1G Linux swap
/dev/sda3   2623488 19400703 16777216    8G Linux filesystem
/dev/sda4  19400704 60549086 41148383 19.6G Linux filesystem

```

### 格式化引导分区

```bash
mkfs.vfat /dev/sda1
```



### 格式化根目录分区

```bash
mkfs.ext4 /dev/sda3 #当然也可以btrfs zfs等等
```

### 激活swap分区

**mkswap**是用来初始化swap分区的命令：

```
mkswap /dev/sda2
```

激活swap分区，使用**swapon**：

```
swapon /dev/sda2
```

使用上面提到的命令创建和激活swap。

### 挂载root分区

```bash
mount /dev/sda3 /mnt/gentoo
```
用 Archiso的小伙伴们这里就使用一条命令搞定：
```bash
genfstab -U /mnt/gentoo >> /mnt/gentoo/etc/fstab
```
## 安装satge包

>"选择一个基础压缩包的系统可以在稍后的安装过程节省大量的时间，特别是当它是一次[选择正确的配置文件](https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Base/zh-cn#.E9.80.89.E6.8B.A9.E6.AD.A3.E7.A1.AE.E7.9A.84.E9.85.8D.E7.BD.AE.E6.96.87.E4.BB.B6)。一个stage包的选择将直接影响未来的系统配置，可以在以后省的头痛。该压缩包 multilib 尽可能使用64位的库，只必要时对32位版本兼容。这对于大多数安装一个很好的选择，因为它在未来的定制提供了极大的灵活性量。那些谁希望自己的系统，能够容易地切换配置,应该下载根据各自的处理器架构 multilib的压缩包选项。"--Gentoo Wiki

**提前更新一下`date`，否则很可能无法下载文件**

### 下载stage压缩包

前往挂载根文件系统的 Gentoo 挂载点：

```bash
cd /mnt/gentoo
```

### 选择要下载的stage包

multilib =>（32和64位）

no-multilib =>（纯64位）

**警告**
>"把一个系统从no-multilib迁移到multilib需要极其丰富的使用Gentoo的知识并熟悉底层的工具链。这一做法甚至可能导致[Toolchain developers](https://wiki.gentoo.org/wiki/Project:Toolchain) 这令人不寒而栗。不适合内心柔弱之人,而且也超出了本指南的范围。" --Gentoo Wiki

### init 系统选择

OpenRC => 较为早古

systemd => 现代的init系统，有很多便利的地方[（如/tmp 如未指定systemd会帮你自动搞上）](https://wiki.archlinux.org/title/tmpfs) 一切看上去都是挺美好的，直到卡进程后(((

```bash
wget http://mirrors.163.com/gentoo/releases/amd64/autobuilds/current-stage3-amd64-desktop-systemd/stage3-amd64-desktop-systemd-1145141919810.tar.xz #以systemd为例，安装时请下载最新的tarball（以tar.gz结尾）
```

```bash
tar xpvf stage3-*.tar.bz2 --xattrs-include='*.*' --numeric-owner
```

>“确保你使用了同样的参数 ( `xpf` 和 `--xattrs-include='*.*'`)。 `x`表示解开（Extract），`v`表示详细信息（Verbose）可以用来查看解压缩时发生了什么（可选参数）， `p` 表示保留权限（Preserve permissions），还有`f` 表示我们要解开一个文件，而不是标准输入。最后，`--numeric-owner` 被用于确保从tarball中提取的文件的用户和组ID与Gentoo发布工程团队预期的保持一致“ ——Gentoo Wiki

### 配置编译选项

*知识点：*

*临时环境变量 =>`export`*

*永久变量 => 如portage会读入`/etc/portage/make.conf`*

## make.conf示例

CFLAGS => GCC

CXXFLAGS => C++

MAKEOPTS =>安装软件的时候同时可以产生并行编译的数目

"第一个设置是标志 `-march=` 和 `-mtune=` ，指定目标体系结构的名称。 可能用到的选项在make.conf.example文件中有描述（作为注释）。 一个常用的值是“native”，它告诉编译器选择当前系统体系结构（用户正在安装Gentoo时的系统）。

第二个是标志 `-O`（即大写的字母O，而不是数字零），它指定了gcc优化级别标志。 可能用到级别的是s（对于大小最优化），0（零 - 无优化），1,2或甚至3等更多的优化选项（每个级别具有与前面相同的标志，加上一些额外选项）。 `-O2`是建议的默认值。 `-O3`在整个系统范围内使用时会导致问题，因此我们建议您坚持使用`-O2`。

另一个普遍使用的优化标记是`-pipe`（不同编译阶段通信使用管道而不是临时文件）。它对产生的代码没有任何影响，但是会使用更多的内存。在内存不多的系统里，gcc可能会被杀掉。如果是那样的话，就不要用这个标记。

使用 `-fomit-frame-pointer`（它将不在寄存器里为不需要帧指针的函数保存帧指针）可能会在调试程序的时候造成严重后果！

尽管这些标志一般在这里默认被定义过，但为了性能最大化，需要分别优化每个程序的这些配置。" ——Gentoo Wiki



**CFLAGS和CXXFLAGS的变量请根据自己的cpu来填写。**

**建议参考[GNU在线手册](https://gcc.gnu.org/onlinedocs/gcc/x86-Options.html#x86-Options) ，[GCC优化](https://wiki.gentoo.org/wiki/GCC_optimization)，`info gcc`,`man make.conf`**

**千 万 不 要 硬 抄**



```bash
# These settings were set by the catalyst build script that automatically
# built this stage.
# Please consult /usr/share/portage/config/make.conf.example for a more
# meowmeowmeow.

COMMON_FLAGS="-march=native -O3 -pipe" #请参照wiki
CFLAGS="${COMMON_FLAGS}"
CXXFLAGS="${COMMON_FLAGS}"
FCFLAGS="${COMMON_FLAGS}"
FFLAGS="${COMMON_FLAGS}"
CPU_FLAGS_X86=""

# NOTE: This stage was built with the bindist Use flag enabled
PORTDIR="/var/db/repos/gentoo"
DISTDIR="/var/cache/distfiles"
PKGDIR="/var/cache/binpkgs"

# This sets the language of build output to English.
# Please keep this setting intact when reporting bugs.
LC_MESSAGES=C
MAKEOPTS="-j8" #根据自己cpu改
GENTOO_MIRRORS="https://mirrors.ustc.edu.cn/gentoo/"
EMERGE_DEFAULT_OPTS="--keep-going --with-bdeps=y --autounmask-write=y --quiet-build=y --jobs=2 -l" #可根据个人喜好改


#ccache一个快速编译器缓存。 
#FEATURES="ccache"
#CCACHE_DIR="/var/cache/ccache" #merge ccache后再取消注释


#aria2，及的看wiki
#FETCHCOMMAND="/usr/bin/aria2c -d \${DISTDIR} -o \${FILE} --allow-overwrite=true --max-tries=5 --max-file-not-found=2 --max-concurrent-downloads=5 --connect-timeout=5 --timeout=5 --split=5 --min-split-size=2M --lowest-speed-limit=20K --max-connection-per-server=9 --uri-selector=feedback \${URI}"
#RESUMECOMMAND="${FETCHCOMMAND}"


ACCEPT_KEYWORDS="~amd64" # ~表示测试版本
ACCEPT_LICENSE="*" # 用"*"表示接受所有许可证 



L10N="en-US zh-CN en zh" # l10n_zh-CN会在桌面系统中使用楷体
LINGUAS="en_US zh_CN en zh"
GRUB_PLATFORMS="efi-64" # 一定要在merge前设置，否则可能会出现grub-efi-amd64-efi-64.efi不存在的错误
INPUT_DEVICES="libinput"



#代理部分
#https_proxy="socks5h://127.0.0.1:7890" 
#http_proxy="socks5h://127.0.0.1:7890"
#ftp_proxy="socks5h://127.0.0.1:7890"

FETCHCOMMAND="curl --retry 3 --connect-timeout 60 --ftp-pasv -Lfo \"\${DISTDIR}/\${FILE}\" \"\${URI}\""
RESUMECOMMAND="curl -C - --retry 3 --connect-timeout 60 --ftp-pasv -Lfo \"\${DISTDIR}/\${FILE}\" \"\${URI}\""
```

<img src="VqZcHU0i.png"  style="zoom:67%;" />

[QAQ^]: 这是Otto,他直接复制了别人的conf，现在编译networkmanager编译部过去力～

------

### **Q&A环节**

王婶：“每次编译时间都要等好久，有什么好办法解决？“

答：柏拉图的《理想国》讲得非常好呢非常建议各位去读一读，以及最近在读《长日将尽》(The Remains of the Day)，也是强推！还有葛亮先生的《北鸢》力推！~~最好的加快编译速度的方法就是升级CPU。~~



赵大爷：“听过配置好Gentoo后机子可以流畅到飞起，让机子延年益寿长生不老永垂不朽是真的吗”

答：

​		~~可以，都可以，有什么不可以的~~

​		专业回答可见[亚晨菊苣的说法](https://www.zhihu.com/question/353429339/answer/2078051783)

​		而且就算是二进制的Gentoo-kernel，效果也未必比配置好的差

​		也许Gentoo高性能这个误会是源于以前硬件发展还没有那么迅猛时，因为 Gentoo 的优化会让老机器上有比较明显的性能优化吧～



你的外国友人Li Hua：“Gentoo安装好复杂，看完这篇还是有点懵，还有没有好用一点的教程/厉害的大佬可以来学习或咨询？”

答：

​		遇到问题，可以在Gentoo-zh群组咨询：

​		[IRC](https://webchat.freenode.net/?channels=gentoo-zh)和[Telegram](https://t.me/gentoo_zh)

​		：



## 安装Gentoo基础系统

### Chrooting
chroot 代表着 change root，切换到根目录
#### Gentoo ebuild 软件仓库
选择镜像的第二个重要步骤是通过/etc/portage/repos.conf/gentoo.conf文件来配置 Gentoo的 ebuild 软件仓库。这个文件包含了更新 Portage 数据库（包含 Portage 需要下载和安装软件包所需要的信息的一个 ebuild 和相关文件的集合）所需要的同步信息。

通过几个简单的步骤就可以完成软件仓库的配置。首先，如果它不存在，则创建repos.conf目录：

```bash
    mkdir --parents /mnt/gentoo/etc/portage/repos.conf
```
接下来，复制 Portage 提供的 Gentoo 仓库配置文件到这个（新创建的）目录：

```bash
cp /mnt/gentoo/usr/share/portage/config/repos.conf /mnt/gentoo/etc/portage/repos.conf/gentoo.conf
```
使用一个文件编辑器或通过使用cat命令来看一眼。文件里的内容应该是.ini格式并且看起来像是这样：
FILE /mnt/gentoo/etc/portage/repos.conf/gentoo.conf(当然也可以自行改)
```bash
[DEFAULT]
main-repo = gentoo
 
[gentoo]
location = /var/db/repos/gentoo
sync-type = rsync
sync-uri = rsync://rsync.gentoo.org/gentoo-portage #这里的rsync.gentoo.org可以换成（mirrors.ustc.edu.cn/portage）
auto-sync = yes
sync-rsync-verify-jobs = 1
sync-rsync-verify-metamanifest = yes
sync-rsync-verify-max-age = 24
sync-openpgp-key-path = /usr/share/openpgp-keys/gentoo-release.asc
sync-openpgp-key-refresh-retry-count = 40
sync-openpgp-key-refresh-retry-overall-timeout = 1200
sync-openpgp-key-refresh-retry-delay-exp-base = 2
sync-openpgp-key-refresh-retry-delay-max = 60
sync-openpgp-key-refresh-retry-delay-mult = 4
```
#### 复制DNS信息
在进行新环境之前，还有一件要做的事情就是复制/etc/resolv.conf中的DNS信息。需要完成这个来确保即使进入到新环境后网络仍然可以使用。/etc/resolv.conf包含着当前网络中的DNS服务器。

要复制这个信息，建议通过cp命令的 --dereference 选项。这可以保障如果/etc/resolv.conf是一个符号链接的话，复制的是那个目标文件而不是这个符号文件自己。否则在新环境中，符号文件将指向一个不存在的文件（因为链接目标非常可能不会在新环境中）。
```bash
cp --dereference /etc/resolv.conf /mnt/gentoo/etc/
```
#### 挂载必要的文件系统
稍等片刻，Linux 的根目录将变更到新的位置。为了确保新环境正常工作，需要确保一些文件系统可以正常使用。

需要提供的文件系统是：

`/proc/` 一个pseudo文件系统（看起来像是常规文件，事实上却是实时生成的），由Linux内核暴露的一些环境信息

`/sys/` 一个pseudo文件系统，像要被取代的/proc/一样，比/proc/更加有结构

`/dev/` 是一个包含全部设备文件的常规文件系统，一部分由Linux设备管理器（通常是udev）管理

`/proc/`位置将要挂载到/mnt/gentoo/proc/，而其它的两个都是绑定挂载。字面上的意思是，例如/mnt/gentoo/sys/事实上就是/sys/（它只是同一个文件系统的第二个条目点），而/mnt/gentoo/proc/是（可以说是）文件系统的一个新的挂载。

```bash
mount --types proc /proc /mnt/gentoo/proc
mount --rbind /sys /mnt/gentoo/sys
mount --make-rslave /mnt/gentoo/sys #如不安装systemd，这个命令可以不要
mount --rbind /dev /mnt/gentoo/dev
mount --make-rslave /mnt/gentoo/dev #如不安装systemd，这个命令可以不要 
```
#### 进入新环境
现在所有的分区已经初始化，并且基础环境已经安装，是时候进入到新的安装环境了。这意思着会话将把根目录（能访问到最顶层的位置）从当前的安装环境（安装CD或其他安装媒介）变为安装系统（叫做初始化分区）。因此叫作 change root 或 chroot。

完成chroot有三个步骤：

使用 chroot 将根目录的位置从 /（在安装媒介里）更改成 /mnt/gentoo/ （在分区里）
使用 source 命令将一些设置（那些在 /etc/profile 中的）重新载入到内存中
更改主提示符来帮助我们记住当前会话在一个 chroot 环境里面。
```bash
chroot /mnt/gentoo /bin/bash
source /etc/profile
export PS1="(chroot) ${PS1}"
```
*如果使用了archiso的，可以arch-chroot /mnt/gentoo /bin/bash*

从现在开始，所有的动作将立即在新Gentoo Linux环境里生效。

#### 挂载 boot 分区
现在已经进入新的环境，必须创建并挂载 boot 分区。 当编译内核并安装引导加载程序时，这将非常重要：
```bash
mount /dev/sda1 /boot
```
#### 配置Portage
从网站安装 Gentoo ebuild 数据库快照
接下来，是安装 Gentoo ebuild 数据库。这个快照包含一组文件，包括通知 `Portage` 中有关可用软件的标题（用于安装），系统管理员可以选择哪些配置文件，软件包或 profile 特定新闻 (news) 项目等。


这将从Gentoo的一个镜像中获取最新的快照（每天发布）并将其安装到系统上：
```bash
emerge-webrsync
```
#### 更新Portage ebuild 数据库
Gentoo 数据库可以更新到最新版本。前面的`emerge-webrsync`命令将安装一个最近的快照（通常是24小时以内），所以这一步是可选的。

假设需要最新更新的软件包（1小时以内），可以使用emerge --sync。这个命令将使用rsync协议来更新 Gentoo ebuild 数据库（之前通过emerge-webrsync获得的）到最新状态。
```bash
emerge --sync
```

**注意：** 如果您的网络不能正常工作，请使用 emerge-webrsync 或是后面用`git`来更新数据库。

### 选择正确的配置文件
配置文件是任何一个Gentoo系统的积木。它不仅指定`USE`、`CFLAGS`和其它重要变量的默认值，还会锁定系统的包版本范围。

使用`eselect`，你能看到当前系统正在使用什么配置文件，现在来使用profile模块：
```bash
eselect profile list
Available profile symlink targets:
  [1]   default/linux/amd64/11.4 *
  [2]   default/linux/amd64/11.4/desktop
  [3]   default/linux/amd64/51.4/desktop/gnome
  [4]   default/linux/amd64/51.1/desktop/kde
```
比如说我要使用`default/linux/amd64/51.4/desktop/kde`，我可以使用`eselect profile set 4`来设置。

接下来，你将要深入接触到Portage了,请务必用`man`来查看里面一些基本的使用方法。

**基础知识：**

`USE`:软件从源代码成为可执行的二进制程序的过程中，需要编译，编译需要依赖，需要优化，需要补丁…这一切的目的是让软件为系统管理员服务。那么系统管理员需要什么？软件最佳运行状态，速度要快，稳定性强，bug尽可能少…`USE`就负责这个，让软件的源代码成为可执行的二进制程序的过程中尽可能的得到定制，最终让系统管理员使用方便、得心应手。我们常使用查看软件 USE 的工具是 equery 这个命令，这个命令可以后面通过 emerge 命令安装 gentoolkit 包实现安装和使用。
`portage` 和 `emerge`:protage 是 Gentoo Linux 的核心，而 emerge 作为它的包管理器。
目录 `package.use`:这个目录里存放自己对每个软件的 USE 自定义，失去分别写，大概的格式保持 <包分类>/<包> <USE flags> 。每个 <USE flags> 用空格分开，如何在终端中查询每个包的 USE flags 呢？使用 equery u <包名> 查看。

### 配置时区
```bash
#openRC
echo "Asia/Shanghai" > /etc/timezone
emerge --config sys-libs/timezone-data

#systemd
ln -sf ../usr/share/zoneinfo/Asia/Shanghai /etc/localtime

echo "en_US.UTF-8 UTF-8
zh_CN.UTF-8 UTF-8" >> /etc/locale.gen

locale-gen
```

### 设置系统语言
```bash
eselect locale list
eselect locale set X
```
请不要先设置为中文，目前的TTY下会乱码
是不是发现`eselect`简直省事省力～

## （可选）优化阶段
这一部分作为可选配置阶段，前提最好是对下列软件有一定了解，可以去翻阅Gentoo wiki上的相关部分，

### 安装 cpuid2cpuflags 软件

```bash
emerge -av --oneshot cpuid2cpuflags
```
然后运行 `cpuid2cpuflags` ，会输出`CPU_FLAGS_X86`的信息，把它们复制后填写到`make.conf`对应位置。

### 重新安装GCC
```bash
emerge -av --oneshot gcc
```
目的是安装最新版本的 gcc，这样才能优化后续更新的软件。特别是如果处理器较新，可能会出现编译错误，所以这个时候需要重新安装。重装完之后，我们就可以改`-march`的参数了。（如笔者使用的是`-march=tigerlake`）

### 安装ccache
```bash
emerge -av --oneshot dev-util/ccache
```
这个软件可以提高编译速度，但是会占用大量的内存，所以我们需要在编译时使用`-j`参数，并且在`make`命令后面加上`-j`参数。

完成安装后，用以下命令创建目录，设置权限：
```bash
mkdir -p /var/cache/ccache
chown root:portage /var/cache/ccache
chmod 2775 /var/cache/ccache
```

编辑我们的 ccache 配置文件 /var/cache/ccache/ccache.conf ，内容如下：
```bash
max_size = 100.0G
umask = 002
cache_dir_levels = 3
```
最后到 make.conf 中把ccache内容的注释取消掉就可以了。
### 安装aria2
```bash
emerge -av --oneshot dev-util/aria2
```
之后去掉/mnt/gentoo/etc/portage/make.conf中关于“aria2”的注释
### 更新@world集合
当系统应用了任何升级，或从 任何profile 构建了stage3 后，应用了变化的 use 标记时，下一步是**必要**的。
```bash
emerge --ask --verbose --update --deep --newuse @world #也就是-aDuvn @world
```
#### 报错处理
真心讲，其实`emerge`已经非常智能了，但是它的更新过程中，会出现很多报错（其实也不算是报错，而是将选择权交给用户），当你看到一片姹紫嫣红又无法解决的时候，不要慌张，也不要心急,多碰壁几次，肯定能解决问题。

**(1)**如果系统提示`/etc`目录下有新的文件，那么我们需要手动更新。

```bash
etc-update --automode -3 #使用时注意查看所写入的更新内容
```

**(2)**没有处理过循环依赖的Gentooer不是好的Gentooer。常见几个如`huffbuzz`和 `freetype `
我们这时候就可以使用临时的USE来解决这个问题。

```bash
USE="-test" emerge -av --update --deep --newuse @world #-test临时从USE中去除
```

**(3)**系统提示需要自行写package.use/package.mask文件，那么我们需要手动写。

什么？还是不能解决？那这边建议先搜索一下，看看是不是有其他的解决方案。实在不行的话，可以上[Telegram](https://t.me/gentoo_zh)或者[IRC](https://webchat.freenode.net/?channels=gentoo-zh)的Gentoo-zh群中讨论。
同时建议使用网络剪切板：

```bash
<命令输出> | curl -F "c=@-" "https://fars.ee/"
```

当没有报错的时候，就可以继续更新了。

### bootstrap
在`/var/db/repos/gentoo/scripts`里面有一个`bootstrap`脚本，它会重新编译emerge包管理器，GCC编译器，llvm编译器，clang编译器，zlib库，glibc库，libtool库等等）
```bash
./bootstrap
```
这个脚本执行完检测后会要求你重新编译安装emerge包管理器，yes同意重新安装。
编译完成后，会提示你需要`emerge -e @world`

使用`env-update`更新环境变量，这样就可以使用新的`emerge`了。

## 配置Linux内核
若是对于新手而言，配置内核将会花上你大量的时间，而且一旦配错了，可能会导致系统无法正常启动。所以非常建议各位先使用`gentoo-kernel-bin`的二进制内核，等自己有空或是进到系统后再使用`gentoo-kernel-source`的源码自己配也不迟。
若你已经捣鼓好config文件后，就可以开始编译了

```bash
make menuconfig
make -jX
make modules_install
make install
```
### 生成initramfs
当自己配置好内核后，想要生成initramfs的话，可以使用`dracut`(为佳)或者`genkernel`。

### 安装微码(zhebufenzanyebuhui)
```bash
emerge -av genkernel sys-firmware/intel-microcode sys-apps/iucode_tool  #for intel
iucode_tool -S 
iucode_tool -S -l /lib/firmware/intel-ucode/* #识别处理器签名并查找相应microcode文件名,会有如“06-9e-0d”这样的编号格式，选择最新的
```
在编译内核时，可以把microcode文件编译进内核，这样就可以在内核启动时加载microcode了。
```bash
Device Driver-->Generic Driver Options-->Firmware loader-->Build name firmware blobs into the kernel binary
```
输入`intel-ucode/06-9e-0d`这样的编号格式，将微码文件直接编译进内核。
**注意**要在`emerge intel-microcode`之前就得在`/etc/portage/make.conf`里将MICROCODE_SIGNATURES设置为"-S"，之后再emerge intel-microcode。

## 安装系统工具
编辑`/etc/conf.d/hostname`，把`HOSTNAME`改成你的主机名
```bash
hostname="ottoqwq-the-gentoo"
 /etc/hosts：
127.0.0.1 localhost
127.0.1.1 ottoqwq-the-gentoo
::1 localhost ip6-localhost ip6-loopback
```
```bash
ip addr   #查看自己的网卡设备信息
```
/etc/conf.d/net：
```bash
config_enp2s0f1="dhcp"  #有线网卡
config_wlo1="dhcp"    #无线wifi网卡
```



/etc/dnsmasq.conf:
```bash
server=114.114.114.114
```

 安装必要的系统日志工具和守护进程工具、文件索引工具、电源管理工具、设备管理工具
```bash
#下面的可以自选安装
emerge --ask app-admin/sysklogd

emerge --ask sys-process/cronie

emerge --ask sys-apps/mlocate

emerge --ask sys-power/acpid

emerge sys-power/thermald

emerge virtual/udev

emerge --oneshot sys-fs/eudev

emerge app-admin/sudo
```
sudo 用户组：
```bash
visudo：
把“%wheel ALL=(ALL) ALL”这一行去掉注释

#设置各用户密码
passwd root
passwd admin
passwd ottoqwq
```
## 配置引导加载程序(GRUB)
```bash
emerge --ask --verbose sys-boot/grub:2
```
UEFI用户注意：运行上述命令将在出现之前输出启用的GRUB_PLATFORMS 值。 当使用支持UEFI的系统时，用户需要确保启用 GRUB_PLATFORMS="efi-64" 参数（默认情况下是这样）。 如果设置不是这样，则需要在安装GRUB2之前将 GRUB_PLATFORMS="efi-64"添加到/etc/portage/make.conf:
```bash
echo 'GRUB_PLATFORMS="efi-64"' >> /etc/portage/make.conf
emerge --ask sys-boot/grub:2
```
### 安装：
```bash
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=grub --recheck
```
### 再一次配置
接下来，基于用户在/etc/default/grub文件和/etc/grub.d中特别配置的脚本文件来生成GRUB2。在大多数场景中，不需要由用户来配置，GRUB2就可以自动检测出哪个内核用于引导（位于/boot/中最高的那一个）以及根文件系统是什么。也可以使用GRUB_CMDLINE_LINUX>变量在/etc/default/grub中附加内核参数。

要生成最终的GRUB2配置，运行grub-mkconfig命令：
```bash
grub-mkconfig -o /boot/grub/grub.cfg
Generating grub.cfg ...
Found linux image: /boot/vmlinuz-4.9.16-gentoo
Found initrd image: /boot/initramfs-genkernel-amd64-4.9.16-gentoo
done
```





**Project still constructing** 
