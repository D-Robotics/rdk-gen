# Development Environment Setup and Compilation Instructions

## Overview

This section introduces the requirements and setup for a cross-compilation development environment, as well as instructions for compiling system images.

## Development Environment

Cross-compilation refers to the process of developing and building software on a host machine and then deploying the built software onto a development board. The host machine generally has higher performance and memory compared to the development board, which can accelerate code builds and allow for the installation of additional development tools, facilitating development.

**Host Compilation Environment Requirements**

It is recommended to use an Ubuntu operating system. If you are using a different system version, adjustments to the compilation environment may be necessary.

For Ubuntu 18.04, install the following packages:

```shell
sudo apt-get install -y build-essential make cmake libpcre3 libpcre3-dev bc bison \
flex python-numpy mtd-utils zlib1g-dev debootstrap \
libdata-hexdumper-perl libncurses5-dev zip qemu-user-static \
curl git liblz4-tool apt-cacher-ng libssl-dev checkpolicy autoconf \
android-tools-fsutils mtools parted dosfstools udev rsync
```

For Ubuntu 20.04, install the following packages:

```shell
sudo apt-get install -y build-essential make cmake libpcre3 libpcre3-dev bc bison \
flex python-numpy mtd-utils zlib1g-dev debootstrap \
libdata-hexdumper-perl libncurses5-dev zip qemu-user-static \
curl git liblz4-tool apt-cacher-ng libssl-dev checkpolicy autoconf \
android-sdk-libsparse-utils android-sdk-ext4-utils mtools parted dosfstools udev rsync
```

For Ubuntu 22.04, install the following packages:

```shell
sudo apt-get install -y build-essential make cmake libpcre3 libpcre3-dev bc bison \
flex python3-numpy mtd-utils zlib1g-dev debootstrap \
libdata-hexdumper-perl libncurses5-dev zip qemu-user-static \
curl repo git liblz4-tool apt-cacher-ng libssl-dev checkpolicy autoconf \
android-sdk-libsparse-utils mtools parted dosfstools udev rsync
```

**Installing the Cross-Compilation Toolchain**

Execute the following command to download the cross-compilation toolchain:

```shell
curl -fO http://archive.d-robotics.cc//toolchain/gcc-arm-11.2-2022.02-x86_64-aarch64-none-linux-gnu.tar.xz
```

Extract and install the toolchain. It is recommended to install it in the /opt directory. Typically, writing to the /opt directory requires sudo permissions. For example:

```shell
sudo tar -xvf gcc-arm-11.2-2022.02-x86_64-aarch64-none-linux-gnu.tar.xz -C /opt
```

Configure the environment variables for the cross-compilation toolchain:

```shell
export CROSS_COMPILE=/opt/gcc-arm-11.2-2022.02-x86_64-aarch64-none-linux-gnu/bin/aarch64-linux-gnu-
export LD_LIBRARY_PATH=/opt/gcc-arm-11.2-2022.02-x86_64-aarch64-none-linux-gnu/lib/x86_64-linux-gnu:$LD_LIBRARY_PATH
export PATH=$PATH:/opt/gcc-arm-11.2-2022.02-x86_64-aarch64-none-linux-gnu/bin/
export ARCH=arm64
```

The above commands set the environment variables temporarily. To make the configuration permanent, you can add these commands to the end of the environment variable files ~/.profile or ~/.bash_profile.

## rdk-gen

`rdk-gen` is used to build a customized operating system image for the D-robotics RDK X3. It provides an extensible framework that allows users to tailor and build the Ubuntu operating system for the RDK X3 according to their specific requirements.

Download the source code:

```shell
git clone https://github.com/D-Robotics/rdk-gen.git
```

After the download is complete, the directory structure of rdk-gen is as follows:

| **Directory**                  | **Description**                                                     |
| ------------------------- | ------------------------------------------------------------ |
| pack_image.sh             | The entry point for building the system image code is:                                       |
| download_samplefs.sh      | Download the pre-built base Ubuntu file system:                       |
| download_deb_pkgs.sh      | Download the D-robotics `.deb` software packages that need to be pre-installed in the system image, including the kernel, multimedia libraries, example code, `tros.bot`, and other components. |
| hobot_customize_rootfs.sh | Customizes and modifies the Ubuntu file system.                               |
| source_sync.sh            | Downloads source code, including bootloader, U-Boot, kernel, example code, etc.      |
| mk_kernel.sh              | Compiles the kernel, device tree, and driver modules.                                   |
| mk_debs.sh                | Generates .deb software packages.                                                |
| make_ubuntu_samplefs.sh   | Creates the Ubuntu system file system; this script can be modified to customize the sample file system.   |
| config                    | Contains content to be placed in the system image's /hobot/config directory. It is a VFAT root partition. If the system boots from an SD card, users can modify this partition's content directly in Windows. |

## Building the System Image

Run the following command to package the system image:

```shell
cd rdk-gen
sudo ./pack_image.sh
```

You need sudo permissions to compile. Upon successful compilation, an *.img system image file will be generated in the deploy directory.

### Overview of the pack_image.sh Compilation Process

1. Calls download_samplefs.sh and download_deb_pkgs.sh scripts to download the sample file system and required .deb packages from D-robotics' file server.
2. Extracts the sample file system and uses the hobot_customize_rootfs.sh script to customize the file system configuration.
3. Installs the .deb packages into the file system.
4. Generates the system image.

## Downloading Source Code

The source code for rdk-linux-related kernel, bootloader, and hobot-xxx software packages is hosted on GitHub. Before downloading the code, please register and log in to GitHub, and add the SSH key for the development server to your user settings via Generating a new SSH key and adding it to the ssh-agent.

The source_sync.sh script is used to download the source code, including bootloader, U-Boot, kernel, example code, etc. This script downloads all the source code locally by executing git clone git@github.com:xxx.git.

Execute the following command to download the main branch code:

```shell
./source_sync.sh -t feat-ubuntu22.04
```

By default, the program will download the source code to the `source` directory:

```
source
├── bootloader
├── hobot-boot
├── hobot-bpu-drivers
├── hobot-camera
├── hobot-configs
├── hobot-display
├── hobot-dnn
├── hobot-dtb
├── hobot-io
├── hobot-io-samples
├── hobot-kernel-headers
├── hobot-multimedia
├── hobot-multimedia-dev
├── hobot-spdev
├── hobot-sp-samples
├── hobot-utils
├── hobot-wifi
└── kernel
```

## kernel

Execute the following command to compile the Linux kernel:

```shell
./mk_kernel.sh
```

After compilation, the kernel image, driver modules, device tree, and kernel header files will be generated in the `deploy/kernel` directory.

```shell
dtb  Image  Image.lz4  kernel_headers  modules
```

These contents will be used by the hobot-boot, hobot-dtb, and hobot-kernel-headers Debian packages. Therefore, if you want to customize these three software packages, you need to compile the kernel first.

## hobot-xxx.deb

hobot-xxx.deb are Debian software packages maintained by D-robotics. After downloading the source code, you can use the mk_deb.sh script to rebuild the Debian packages.

The help information is as follows:

```shell
$ ./mk_debs.sh help
The debian package named by help is not supported, please check the input parameters.
./mk_deb.sh [all] | [deb_name]
    hobot-multimedia-dev, Version 2.0.0
    hobot-wifi, Version 2.0.0
    hobot-camera, Version 2.0.0
    hobot-dtb, Version 2.0.0
    hobot-configs, Version 2.0.0
    hobot-io, Version 2.0.0
    hobot-spdev, Version 2.0.0
    hobot-boot, Version 2.0.0
    hobot-sp-samples, Version 2.0.0
    hobot-bpu-drivers, Version 2.0.0
    hobot-multimedia-samples, Version 2.0.0
    hobot-dnn, Version 2.0.0
    hobot-io-samples, Version 2.0.0
    hobot-kernel-headers, Version 2.0.0
    hobot-utils, Version 2.0.0
    hobot-multimedia, Version 2.0.0
    hobot-display, Version 2.0.0
```

### Full Build

Execute the following command to rebuild all the Debian packages (ensure that the kernel has been compiled first):

```shell
./mk_deb.sh
```

After the build is complete, the `.deb` packages will be generated in the `deploy/deb_pkgs` directory.

### Building Individual Packages

The mk_deb.sh script supports building specific packages individually by providing the package name as a parameter during execution. For example:

```shell
./mk_deb.sh hobot-configs
```

## bootloader

The bootloader source code is used to generate a minimal boot image, miniboot.img, which includes the partition table, SPL, DDR, BL31, and U-Boot in a single boot firmware.

The minimal boot image for the RDK X3 is typically maintained and released by D-robotics,You can download the corresponding version from (http://archive.d-robotics.cc/downloads/miniboot/).

Follow these steps to recompile and generate miniboot:

### Sync U-Boot Code

Execute the following command to download the U-Boot source code:

```shell
git submodule init
git submodule update
```

### Select the hardware configuration file:

```shell
cd build
./xbuild.sh lunch

You're building on #221-Ubuntu SMP Tue Apr 18 08:32:52 UTC 2023
Lunch menu... pick a combo:
      0. horizon/x3/board_ubuntu_emmc_sdcard_config.mk
      1. horizon/x3/board_ubuntu_emmc_sdcard_samsung_4GB_config.mk
      2. horizon/x3/board_ubuntu_nand_sdcard_config.mk
      3. horizon/x3/board_ubuntu_nand_sdcard_samsung_4GB_config.mk
Which would you like? [0] :  
```

Select the board-level configuration file according to the prompt.

The pre-configured files are designed for different hardware configurations of development boards, varying in aspects such as whether miniboot is burned to eMMC or NAND, the DDR model and capacity, and the root file system:

| Board-Level Configuration File                                   | Memory               | rootfs       | Minimum Boot Image Storage | Main Storage    |
| ---------------------------------------------- | ------------------ | ------------ | ------------------ | ----------- |
| board_ubuntu_emmc_sdcard_config.mk             | LPDDR4 2GB | ubuntu-20.04 | emmc               | sdcard      |
| board_ubuntu_emmc_sdcard_samsung_4GB_config.mk | LPDDR4 4GB | ubuntu-20.04 | emmc               | sdcard      |
| board_ubuntu_nand_sdcard_config.mk             | LPDDR4 2GB | ubuntu-20.04 | nand               | sdcard/emmc |
| board_ubuntu_nand_sdcard_samsung_4GB_config.mk | LPDDR4 4GB | ubuntu-20.04 | nand               | sdcard/emmc |

**Minimum Boot Image Storage:** The storage where miniboot is burned. Users of RDK X3 and RDK X3 Module should choose the NAND flash method.

**Main Storage: ** The storage for the Ubuntu system image. SD cards and eMMC are interchangeable, meaning that an image burned to a microSD card can also be burned to eMMC.



The `lunch` command also supports specifying a number and board-level configuration file name to complete the configuration directly.

```shell
$ ./xbuild.sh lunch 2

You're building on #221-Ubuntu SMP Tue Apr 18 08:32:52 UTC 2023
You are selected board config: horizon/x3/board_ubuntu_nand_sdcard_config.mk

$ ./xbuild.sh lunch board_ubuntu_nand_sdcard_config.mk

You're building on #221-Ubuntu SMP Tue Apr 18 08:32:52 UTC 2023
You are selected board config: horizon/x3/board_ubuntu_nand_sdcard_config.mk
```

### Full Build

Navigate to the `build` directory and execute `xbuild.sh` to perform the overall build:

```shell
cd build
./xbuild.sh
```

After a successful build, the following image files will be generated in the output directory (e.g., deploy_ubuntu_xxx):
miniboot.img
uboot.img
disk_nand_minimum_boot.img
Among these, disk_nand_minimum_boot.img is the minimal boot image file.

### Modular Compilation

Compile individual modules using the xbuild.sh script. The resulting image files will be output to the build output directory (e.g., deploy_ubuntu_xxx).

```shell
./xbuild.sh miniboot | uboot
```

**miniboot：** use mk_miniboot.sh get miniboot.img

**uboot:**  use mk_uboot.sh get uboot.img

After modular compilation, you can use the pack command to package the disk_nand_minimum_boot.img. 

```shell
./xbuild.sh pack
```

## Ubuntu File System Creation

This section describes how to create the samplefs_desktop-v3.0.0.tar.gz file system. D-robotics maintains this file system, but if you have customization needs, you will need to recreate it according to the instructions in this chapter.

### Environment Setup

It is recommended to use an Ubuntu host for creating the Ubuntu file system for the development board. First, install the following packages in the host environment:

```shell
sudo apt-get install wget ca-certificates device-tree-compiler pv bc lzop zip binfmt-support \
build-essential ccache debootstrap ntpdate gawk gcc-arm-linux-gnueabihf qemu-user-static \
u-boot-tools uuid-dev zlib1g-dev unzip libusb-1.0-0-dev fakeroot parted pkg-config \
libncurses5-dev whiptail debian-keyring debian-archive-keyring f2fs-tools libfile-fcntllock-perl \
rsync libssl-dev nfs-kernel-server btrfs-progs ncurses-term p7zip-full kmod dosfstools \
libc6-dev-armhf-cross imagemagick curl patchutils liblz4-tool libpython2.7-dev linux-base swig acl \
python3-dev python3-distutils libfdt-dev locales ncurses-base pixz dialog systemd-container udev \
lib32stdc++6 libc6-i386 lib32ncurses5 lib32tinfo5 bison libbison-dev flex libfl-dev cryptsetup gpg \
gnupg1 gpgv1 gpgv2 cpio aria2 pigz dirmngr python3-distutils distcc git dos2unix apt-cacher-ng
```

### Tools Introduction

#### debootstrap

debootstrap is a tool for Debian/Ubuntu systems used to create a basic system (root file system). The generated directory conforms to the Linux Filesystem Hierarchy Standard (FHS), including directories like /boot, /etc, /bin, /usr, etc. However, it is much smaller than a full Linux distribution and has limited functionality, serving only as a "basic system" that can be customized to meet specific needs.

Installing debootstrap on Ubuntu (PC)

```shell
sudo apt-get install debootstrap
```

Usage method

```shell
# Can add parameters to specify the source
sudo debootstrap [options] <suite> <target> [mirror]
```

#### chroot

chroot，Change root directory. In Linux systems, the default directory structure starts with '/', which is the root. After using chroot, the system's directory structure will use the specified location as the `/` position.

#### parted

parted is a powerful disk partitioning and partition resizing tool developed by the GNU organization. Unlike fdisk, it supports resizing partitions. As a tool designed for Linux, it is not built to handle multiple partition types associated with fdisk, but it can handle the most common partition formats, including ext2, ext3, fat16, fat32, NTFS, ReiserFS, JFS, XFS, UFS, HFS, and Linux swap partitions.

### Creating Ubuntu rootfs script code

Execute the following command to generate the Ubuntu file system:

build desktop ubuntu
```shell
cd samplefs
sudo ./make_ubuntu_rootfs.sh
```

The output result of successful compilation:

```shell
desktop/                                   # compile output directory
├── jammy-xj3-arm64                        # After successful compilation, the generated root file system will have a large number of temporary system files
├── samplefs_desktop-v3.0.0.tar.gz         # Compress and package the required content in jammy-xj3-arm64
└── samplefs_desktop-v3.0.0.tar.gz.info    # Which apt packages are currently installed on the system
```

build server ubuntu
```shell
cd samplefs
sudo ./make_ubuntu_rootfs.sh server
```

The output result of successful compilation:

```shell
server/                                   # compile output directory
├── jammy-xj3-arm64                       # After successful compilation, the generated root file system will have a large number of temporary system files
├── samplefs_server-v3.0.0.tar.gz         # Compress and package the required content in jammy-xj3-arm64
└── samplefs_server-v3.0.0.tar.gz.info    # Which apt packages are currently installed on the system
```

After decompressing samplefs_desktop-v3.0.0.tar.gz or samplefs_server-v3.0.0.tar.gz should be included

```shell
rootfs/
├── app
├── bin -> usr/bin
├── boot
├── dev
├── etc
├── home
├── lib -> usr/lib
├── media
├── mnt
├── opt
├── proc
├── root
├── run
├── sbin -> usr/sbin
├── srv
├── sys
├── tmp
├── userdata
├── usr
└── var

21 directories, 5 files
```

### Customized modifications

Definition of key variables in the code:

**PYTHON_PACKAGE_LIST**： Installed Python package

**DEBOOTSTRAP_LIST**：The Debian package installed during the execution of Bootstrap

**BASE_PACKAGE_LIST**： The most basic Debian package required for UBuntu system installation

**SERVER_PACKAGE_LIST**：Server versions of Ubuntu systems will install additional Debian packages on top of the base version

**DESKTOP_PACKAGE_LIST**: Software packages that need to be installed to support desktop graphical interfaces

The 'samplefs_desktop' file system maintained by D-robotics will contain the contents of all the configuration packages mentioned above, and users can add or delete them according to their own needs.
