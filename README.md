# nanopi-r6s
本仓库用于存储 NanoPi R6S 开发套件的 Linux 系统固件，包括 U-Boot、Kernel、rootfs 等资源，以及相关构建工具。

主要内容源自FriendlyARM 官方提供的 [sd-fuse_rk3588](https://github.com/friendlyarm/sd-fuse_rk3588)、[kernel-rockchip](https://github.com/friendlyarm/kernel-rockchip)，并在此基础上进行调整，以便于适用于 NanoPi R6S 套件。同时对 Kernel 和文件系统进行了一些修改，以满足本人的使用需求。

关于 NanoPi R6S 的更多内容，可以参考《[NanoPi R6S 开发教程](https://getiot.tech/category/nanopir6s)》。



## 简介

sd-fuse 是一系列工具和脚本的集合, 用于固件的打包, 具体用途如下：

* 制作分区映象文件, 例如将 rootfs 目录打包成 rootfs.img；
* 将多个分区映象文件打包成可直接写 SD 卡的单一映象文件；
* 简化内核和 uboot 的编译, 一键编译内核、第三方驱动, 并更新 rootfs.img 中的内核模块。
  



## 运行环境

* 在电脑主机端使用
* 推荐的操作系统：Ubuntu 18.04 及以上 64 位操作系统
* 推荐运行此脚本初始化开发环境: https://github.com/friendlyarm/build-env-on-ubuntu-bionic



## 支持的目标板 OS

* buildroot
* debian-buster-desktop-arm64
* debian-bullseye-desktop-arm64
* ubuntu-jammy-desktop-arm64
* ubuntu-jammy-minimal-arm64
* friendlywrt22
* friendlywrt22-docker
* friendlywrt21
* friendlywrt21-docker


这些 OS 名称是分区映象文件存放的目录名，在脚本内亦有严格定义，所以不能改动，例如要制作 ubuntu-jammy-desktop 的 SD 固件, 可使用如下命令：
```bash
./mk-sd-image.sh ubuntu-jammy-desktop-arm64
```



## 获得打包固件所需要的素材

制作固件所需要的素材分为几种:
* 内核和uboot源代码: 在[网盘](https://dl.friendlyelec.com/nanopir6s)的 "07_源代码" 目录中
* 分区映象文件: 在[网盘](https://dl.friendlyelec.com/nanopir6s)的 "03_分区镜像文件" 目录中
* 文件系统压缩包: 在[网盘](https://dl.friendlyelec.com/nanopir6s)的 "06_文件系统" 目录中
如果没有提前准备好素材文件, 脚本亦会自动下载所需要的素材, 不过因为http服务器带宽的关系, 速度可能会较慢。



## 脚本功能

| 脚本文件            | 功能描述                         |
| ------------------- | -------------------------------- |
| fusing.sh           | 将镜像烧写至 SD 卡               |
| mk-sd-image.sh      | 制作 SD 卡映象                   |
| mk-emmc-image.sh    | 制作 eMMC 卡刷固件（SD-to-eMMC） |
| build-boot-img.sh   | 打包 boot.img                    |
| build-rootfs-img.sh | 打包文件系统映象 rootfs.img      |
| build-kernel.sh     | 编译内核，或内核头文件           |
| build-uboot.sh      | 编译 uboot                       |



## 如何使用
### 重新打包 SD 运行固件
*注：这里以 ubuntu-jammy-desktop 系统为例进行说明*

下载本仓库到本地，然后下载并解压 ubuntu-jammy-desktop 系统的分区映象文件压缩包，由于 http 服务器带宽的关系，wget 命令可能会比较慢，推荐从网盘上下载同名的文件：

```bash
git clone https://github.com/friendlyarm/sd-fuse_rk3588 -b master sd-fuse_rk3588-master
cd sd-fuse_rk3588-master
wget http://112.124.9.243/dvdfiles/rk3588/images-for-eflasher/ubuntu-jammy-desktop-arm64-images.tgz
tar xvzf ubuntu-jammy-desktop-arm64-images.tgz
```
解压后，会得到一个名为 ubuntu-jammy-desktop-arm64 的目录，可以根据项目需要，对目录里的文件进行修改，例如把 rootfs.img 替换成自已修改过的文件系统映象，或者自已编译的内核和 uboot 等，准备就绪后，输入如下命令将系统映像写入到 SD 卡（其中 /dev/sdX 是你的 SD 卡设备名）：
```bash
sudo ./fusing.sh /dev/sdX ubuntu-jammy-desktop-arm64
```
或者，打包成可用于 SD 卡烧写的单一映象文件：
```bash
./mk-sd-image.sh ubuntu-jammy-desktop-arm64
```
命令执行成功后，将生成以下文件，此文件可烧写到 SD 卡运行：
```bash
out/rk3588-sd-ubuntu-jammy-desktop-5.10-arm64-YYYYMMDD.img
```



### 重新打包 SD-to-eMMC 卡刷固件

*注：这里以 ubuntu-jammy-desktop 系统为例进行说明*

下载本仓库到本地，然后下载并解压分区映象压缩包，这里需要下载 ubuntu-jammy-desktop 和 eflasher 系统的文件：

```bash
git clone https://github.com/friendlyarm/sd-fuse_rk3588 -b master sd-fuse_rk3588-master
cd sd-fuse_rk3588-master
wget http://112.124.9.243/dvdfiles/rk3588/images-for-eflasher/ubuntu-jammy-desktop-arm64-images.tgz
tar xvzf ubuntu-jammy-desktop-arm64-images.tgz
wget http://112.124.9.243/dvdfiles/rk3588/images-for-eflasher/emmc-flasher-images.tgz
tar xvzf emmc-flasher-images.tgz
```
再使用以下命令，打包卡刷固件，autostart=yes 参数表示使用此固件开机时，会自动进入烧写流程：
```bash
./mk-emmc-image.sh ubuntu-jammy-desktop-arm64 autostart=yes
```
命令执行成功后，将生成以下文件，此文件可烧写到 SD 卡运行：
```bash
out/rk3588-eflasher-ubuntu-jammy-desktop-5.10-arm64-YYYYMMDD.img
```



### 定制文件系统

*注：这里以 ubuntu-jammy-desktop 系统为例进行说明*

下载本仓库到本地，然后下载并解压分区映象压缩包：

```bash
git clone https://github.com/friendlyarm/sd-fuse_rk3588 -b master sd-fuse_rk3588-master
cd sd-fuse_rk3588-master
wget http://112.124.9.243/dvdfiles/rk3588/images-for-eflasher/ubuntu-jammy-desktop-arm64-images.tgz
tar xvzf ubuntu-jammy-desktop-arm64-images.tgz
```
下载对应系统的文件系统压缩包并解压，这里需要注意的是，由于权限问题，解压时需要加上 sudo：
```bash
wget http://112.124.9.243/dvdfiles/rk3588/rootfs/rootfs-ubuntu-jammy-desktop-arm64.tgz
sudo tar xzf rootfs-ubuntu-jammy-desktop-arm64.tgz
```
可以根据需要，对文件系统目录进行更改，例如：
```bash
sudo sh -c 'echo hello > ubuntu-jammy-desktop-arm64/rootfs/root/welcome.txt'
```
然后用以下命令将文件系统目录打包成 rootfs.img：
```bash
sudo ./build-rootfs-img.sh ubuntu-jammy-desktop-arm64/rootfs ubuntu-jammy-desktop-arm64
```
打包成 SD 卡映象文件：
```bash
./mk-sd-image.sh ubuntu-jammy-desktop-arm64
```
#### 文件系统 Tips
* 可利用 debootstrap 工具对文件系统进行定制，预装软件包等
* [如何从设备上导出文件系统?](README.md)

### 编译内核
*注：这里以 ubuntu-jammy-desktop 系统为例进行说明*

下载本仓库到本地，然后下载并解压分区映象压缩包：

```bash
git clone https://github.com/friendlyarm/sd-fuse_rk3588 -b master sd-fuse_rk3588-master
cd sd-fuse_rk3588-master
wget http://112.124.9.243/dvdfiles/rk3588/images-for-eflasher/ubuntu-jammy-desktop-arm64-images.tgz
tar xvzf ubuntu-jammy-desktop-arm64-images.tgz
```
从 github 克隆内核源代码到本地，用环境变量 `KERNEL_SRC` 来指定本地源代码目录：
```bash
export KERNEL_SRC=$PWD/kernel
git clone https://github.com/friendlyarm/kernel-rockchip -b nanopi5-v5.10.y_opt --depth 1 ${KERNEL_SRC}
```
根据需要配置内核：
```bash
cd $KERNEL_SRC
touch .scmversion
make ARCH=arm64 nanopi6_linux_defconfig
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- menuconfig     # 根据需要改动配置
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- savedefconfig
cp defconfig ./arch/arm64/configs/my_defconfig                  # 保存配置 my_defconfig
git add ./arch/arm64/configs/my_defconfig
cd -
```
使用 `KCFG` 环境变量指定内核的配置（`KERNEL_SRC` 指定源代码目录），使用你的配置编译内核：
```bash
export KERNEL_SRC=$PWD/kernel
export KCFG=my_defconfig
./build-kernel.sh ubuntu-jammy-desktop-arm64
```

#### 编译内核头文件
设置环境变量 `MK_HEADERS_DEB` 为 1，将编译内核头文件：
```bash
MK_HEADERS_DEB=1 ./build-kernel.sh ubuntu-jammy-desktop-arm64
```
#### 其他
* 设置环境变量 `BUILD_THIRD_PARTY_DRIVER` 为 1 将跳过第三方驱动模块的编译

### 编译 u-boot
*注：这里以 ubuntu-jammy-desktop 系统为例进行说明*

下载本仓库到本地，然后下载并解压分区映象压缩包：

```bash
git clone https://github.com/friendlyarm/sd-fuse_rk3588 -b master sd-fuse_rk3588-master
cd sd-fuse_rk3588-master
wget http://112.124.9.243/dvdfiles/rk3588/images-for-eflasher/ubuntu-jammy-desktop-arm64-images.tgz
tar xvzf ubuntu-jammy-desktop-arm64-images.tgz
```
从 github 克隆与 OS 版本相匹配的 u-boot 源代码到本地，环境变量 `UBOOT_SRC` 用于指定本地源代码目录：
```bash
export UBOOT_SRC=$PWD/uboot
git clone https://github.com/friendlyarm/uboot-rockchip -b nanopi6-v2017.09 --depth 1 ${UBOOT_SRC}
./build-uboot.sh ubuntu-jammy-desktop-arm64
```



## Q & A

### 如何查询 SD 卡的设备文件名

在未插入 SD 卡的情况下输入：
```bash
ls -1 /dev > ~/before.txt
```
插入 SD 卡，输入以下命令查询：
```bash
ls -1 /dev > ~/after.txt
diff ~/before.txt ~/after.txt
```
