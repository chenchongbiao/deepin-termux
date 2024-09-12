# 介绍

之前介绍了，关于小米平板5安装 deepin 23 的内容，不过需要解 BL 锁和获取设备的 root 权限，限制条件过多，本次分享下关于 Termux 中使用 deepin 23 (arm64)系统，不需要刷机和获取 root 权限。安装 Termux 应用进入后，会提示是否支持运行。

# 安装环境

## 安装软件

### Termux

Termux 是一个 Android 终端应用程序和 Linux 环境。

[项目地址](https://github.com/termux/termux-app)

[Github 下载链接](https://github.com/termux/termux-app/releases/download/v0.118.1/termux-app_v0.118.1+github-debug_arm64-v8a.apk)

也可以选择第三方的 ZeroTermux

[项目地址](https://github.com/hanxinhao000/ZeroTermux)

[Github 下载链接](https://github.com/hanxinhao000/ZeroTermux/releases/download/release/ZeroTermux-0.118.1.40.apk)

### Termux:X11

Termux X11 服务器附加应用程序。用来显示 Termux 上运行的桌面环境。

[项目地址](https://github.com/termux/termux-x11)

[Github 下载链接](https://github.com/termux/termux-x11/releases/download/nightly/app-arm64-v8a-debug.apk)

下载两个安卓应用，并安装。打开 Termux 之后能显示出终端说明安装成功。

## 配置基础软件

更新软件源

```bash
pkg update
```

### ssh

安装 ssh，方便远程收手机上调试。

```bash
pkg install openssh
```

输入 `sshd` 开启服务。

输入 `whoami` 查看用户名。如 u0_a216

输入 `ifconfig` 查看 IP 地址。如 192.168.2.103

Termux 上 ssh 端口是 8022

用 passwd 设置密码。输入两次，输入密码时不会显示。

使用 ssh 工具远程。

或者在 PC 系统的终端上使用命令行

```bash
ssh u0_a216@192.168.2.103 -p 8022
```

## 安装 proot-distro

这是一个基于proot实用程序的容器环境管理器，能够模拟chroot和mount --bind 。可以使用 proot 运行 deepin 的系统，不过由于 dde 桌面会依赖 systemd， proot 中无法使用 systemd，该方法只能使用 deepin 的命令行界面。

```bash
pkg install proot-distro
```

安装 deepin

```bash
proot-distro install deepin
```

proot-distro 会从 Github 上下载根文件系统。如果下载失败，可以手动[下载根文件系统](https://github.com/termux/proot-distro/releases/download/v4.6.0/deepin-aarch64-pd-v4.6.0.tar.xz)(需要一点魔法)。

ps: proot-distro 4.16 版本中的 deepin 没有做 usr 和合并，dbus 等包无法安装。
已提交修复 https://github.com/termux/proot-distro/pull/447，请在后续版本安装使用。

123盘链接

拷贝 deepin-rootfs.zip 到 `/data/data/com.termux/files/usr/var/lib/proot-distro/installed-rootfs/`

```bash
unzip deepin-rootfs.zip
tar Jxvf deepin-aarch64-pd-v4.16.0.tar.xz
```

重命名

```
mv  deepin-aarch64 deepin
```

登录 deepin

```bash
proot-distro login deepin
```

ps: 使用了手动下载根文件系统的方式，需要手动修改下 DNS 服务器的信息。

```
echo "nameserver 8.8.8.8" > /etc/resolv.conf
```

安装需要的包

```bash
apt update -y
apt install sudo vim adduser apt-utils
```

创建用户，输入两次密码，密码不显示，剩下的选项回车即可。

```bash
adduser deepin
```

添加 sudo 权限，使用 `visudo`，找到 `root    ALL=(ALL:ALL) ALL`，找到

```bash
deepin ALL=(ALL:ALL) ALL
```

`su - deepin` 切换 deepin 用户。

或者 `proot-distro login deepin --user deepin`

## 安装 QEMU

QEMU 可以看成一款虚拟机，他可以模拟很多CPU架构。比如Alpha, ARM, Cris, i386, M68K, PPC, Sparc, Mips等；以及大部分的硬件设备，也就可以模拟出不同的目标系统。

proot 运行的并非完整系统，想要获得更完整的系统体验可以尝试 QEMU。后续可以安装桌面环境后，可以通过 VNC 访问。

需要安装 qemu-system-aarch64

```bash
pkg install x11-repo
pkg update
pkg install qemu-system-aarch64
```

需要严格遵守操作顺序，`x11-repo` 安装后将会新增引入x11、qemu等相关软件仓库，若不安装此包并update仓库源，将无法找到可安装的 `qemu-system-aarch64` 包

制作磁盘镜像。

```bash
git clone git@github.com:chenchongbiao/deepin-arm64-img.git
```

```bash
cd deepin-arm64-img
./build-img.sh
```

该脚本需要在支持 qemu-user-static 支持 arm64 的 PC 系统环境下运行，或者直接在 arm64 架构的设备上执行， 脚本定义的软件包及命令不适用 Termux。

如果不想自定义构建可以下载制作好的磁盘文件。系统使用的UEFI 启动需要额外指定一个UEFI 固件。

UEFI 固件

[123盘](https://www.123pan.com/s/kzYMjv-OYmVH) 提取码:yfP2

磁盘文件

[123盘](https://www.123pan.com/s/kzYMjv-RYmVH) 提取码:mRYg

将下载好的文件使用USB线连接到 PC 系统上或者通过 scp 等命令传到 Termux 上。

```bash
xz -d deepin-arm64.img.xz
```

解压后执行命令。以下命令在支持 qemu-system-aarch64 的设备上也同样适用，可以在 PC 系统上验证完后，拷贝到移动设备上。

```bash
qemu-system-aarch64 \
    -machine virt \
    -m 4G \
    -smp 2,cores=2,threads=1,sockets=1 \
    -cpu cortex-a76 \
    -drive file=AAVMF_CODE.ms.fd,if=pflash,format=raw,readonly=on \
    -drive file=deepin-arm64.img,if=none,id=hd0,format=raw \
    -device virtio-blk-pci,drive=hd0 \
    -device qemu-xhci \
    -netdev user,id=mynet \
    -device virtio-net-pci,netdev=mynet \
    -vnc 0.0.0.0:1 \
    -device virtio-balloon-pci \
    -device virtio-gpu-pci \
    -device usb-tablet,id=tablet \
    -device usb-kbd \
    -serial mon:stdio
```

-m 4G 参数表示分配给虚拟机4GB 的内存，可以根据设备上的实际内存分配，需要小于实际大小。

smp 2,cores=2,threads=1,sockets=1 配置虚拟机具有两个CPU核心，每个核心只有一个线程，并且这些核心位于一个物理插槽（socket）上。根据设备商支持的核心数分配，需要小于实际核心数。

-cpu cortex-a76 指定虚拟机的CPU类型为Cortex-A76，这是一种高性能的ARM v8架构的CPU，可以通过 qemu-system-aarch64 -cpu help 查看支持的 CPU 类型修改。

-drive file=AAVMF_CODE.ms.fd,if=pflash,format=raw,readonly=on 加载名为 `AAVMF_CODE.ms.fd` 的文件作为只读的PFlash设备，通常用于存储UEFI固件。

-drive file=deepin-arm64.img,if=none,id=hd0,format=raw 加载名为 `deepin-arm64.img`的磁盘映像文件，并将其标识为 `hd0`，格式为 `raw`。

-vnc 0.0.0.0:1 启用VNC服务器，监听所有网络接口（`0.0.0.0`）上的端口1（通常指的是5901）。这使得可以通过VNC客户端远程访问虚拟机的图形界面。后续安装桌面环境后，可以使用 VNC 客户端连接。

终端上会输出启动的日志，根据设备的性能高低，启动时间不同，以实际为准。

一直到以下界面说明启动成功，输入 root，默认密码为空。

```bash
Deepin GNU/Linux 23 deepin-arm64 ttyAMA0

deepin-arm64 login: 
```

到这里 Termux 启动 deepin arm64 系统的内容就告一段落。

在命令行界面输入 `poweroff` 关闭系统。

### 扩容磁盘文件

一开始的磁盘文件只分配了 2G, 需要进行扩容。

扩容前需要安装工具。

```bash
pkg update
pkg install qemu-utils
```

为了后续，安装 dde 桌面环境，完整大约8-9G，这里为了保证后续使用，添加18G，可以根据实际使用调整。

```bash
qemu-img resize deepin-arm64.img +18G
```

通过 `ls -lh deepin-arm64.img` 命令查看磁盘文件已经显示为 20 G了，不过还需要给分区进行扩容。

需要安装 parted 工具，parted 是一个在 Linux 和其他类 Unix 操作系统中用于创建、删除、检查、复制分区表以及对硬盘等存储设备进行分区的命令行工具。

```bash
pkg install parted
```

```bash
parted deepin-arm64.img
```

先输入 `print` 查看分区。

进入 Parted 的界面，会提示是否修复分区，输入 Fix 进行修复。

会输出两个分区，第 15 分区为 efi 分区，第 1 分区为根分区，我们需要将前面扩容的大小分给根分区。

```bash
resizepart 1 100%
```

没有输出说明执行正常，再用 `print` 查看，根分区已经被扩容了。

q 退出分区工具。

PS: 需要修复文件系统，不过在移动设备上没有 root 权限，无法进行磁盘挂载，需要借助 PC 系统上的 e2fsck 命令修复扩容后的文件系统，不进行修复也行，每次进入系统后会进行一次文件系统修复。如果需要修复请到 PC 系统上，进行操作。

```bash
sudo apt install e2fsprogs
sudo losetup -Pf deepin-arm64.img
```

`lsblk` 查看挂载的 loop 设备。

修复文件系统

```bash
sudo e2fsck -f /dev/loop0p1
```

`loop0` 根据实际设备修改。

卸载设备

```bash
sudo losetup -D /dev/loop0
```

在 PC 系统上修复完毕后，拷贝回移动设备。

输入前面启动虚拟机的命令，重新进入系统。

`df -h` 可以看到系统已经被正确扩容过了。

### 添加用户

该步骤不支持跳过，如不创建非 root 用户则无法正常启动 lightdm.service

需要添加个 deepin 用户。加入到 sudo 用户组。

```bash
useradd -m -g users deepin && usermod -a -G sudo deepin
```

设置默认 shell。

```bash
chsh -s /bin/bash deepin
```

设置密码 为 deepin。

```bash
echo deepin:deepin | chpasswd
```

### 尝试运行 dde 桌面

切换 deepin 用户。

```bash
su - deepin
```

如果需要安装 dde 桌面环境。

```bash
sudo apt update
sudo apt install deepin-desktop-environment-base \
    deepin-desktop-environment-cli \
    deepin-desktop-environment-core \
    deepin-desktop-environment-extras
```

遇到 Configuring console-setup 配置，选择 27。

遇到 Character set to support: 配置，选择 23。

安装软件包过多，需要等待。

安装完成后，重启保证一些更改生效。

移动设备上安装 RVNC Viewer 或者其他 VNC 客户端，创建连接填写 localhost:5901。

进入桌面大家都可以尝试一下。

镜像地址：
