# ZCU102_petalinux

## Introduction
How to make petalinux on ZCU102 board

## Getting Started
### Install packages
```
$ sudo apt install -y python gawk gcc git make net-tools libncurses5-dev libncursesw5-dev zlib1g:i386 libssl-dev flex bison libselinux1 gnupg wget diffstat chrpath socat xterm autoconf libtool tar unzip texinfo zlib1g-dev gcc-multilib build-essential libsdl1.2-dev libglib2.0-dev screen pax gzip avrdude tftp tftpd-hpa tftp-server vim
$ sudo apt autoremove -y
$ sudo service tftpd-hpa status

$ sudo mkdir -p /tools/Xilinx/PetaLinux/2018.3
$ sudo chmod -R o+x /tools	
(dir: 775, file: 644)
$ sudo chown -R $(whoami):$(whoami) /tools
$ cd /tools/Xilinx/PetaLinux/2018.3

$ mkdir ~/project_5_petalinuxBase
$ cp ~/Downloads/design_1_wrapper.hdf ~/project_5_petalinuxBase

$ mkdir ~/build_results

$ chmod +x ~/Downloads/petalinux-v2018.3-final-installer.run
$ ~/Downloads/petalinux-v2018.3-final-installer.run /tools/Xilinx/PetaLinux/2018.3
```
copy design_1_wrapper.hdf to project_5_petalinuxBase

### Configure the Petalinux project with the hardware description file
```
$ source settings.sh
$ petalinux-create --type project --template zynqMP --name zcu102-petalinuxbase
$ petalinux-create --type project --source ~/Downloads/xilinx-zcu102-zu9-es2-rev1.0-v2018.3-final.bsp --name zcu102-petalinuxbase
$ cd zcu102-petalinuxbase
$ petalinux-config --get-hw-description=~/project_5_petalinuxBase
```

```
Enable 'Image Packaging Configuration > Root file system type > SD card'   
Disable 'DTG Settings > Kernel Bootargs > [] generate boot args automatically'  
Set following kernel bootargs
> earlycon clk_ignore_unused earlyprintk root=/dev/mmcblk0p2 rw rootwait cma=1024M
or
> earlycon console=ttyPS0,115200 clk_ignore_unused root=/dev/mmcblk0p2 rw rootwait
or
> earlycon earlyprintk clk_ignore_unused root=/dev/mmcblk0p2 rw rootwait uio_pdrv_genirq.of_id=generic-uio cma=1024M rootfstype=ext3 init=/bin/sh stdout-path=serial0:115200n8
```

Save and Exit the Petalinux project configuration

```
$ petalinux-build -c bootloader -x distclean
```

### Before kernel configuration
#### ImportError: cannot import name '_gi'
```
$ sudo apt-get install screen
$ vi /tools/Xilinx/PetaLinux/2018.3/zcu102-petalinuxbase/project-spec/meta-user/conf/petalinuxbsp.conf

#OE_TERMINAL = "tmux" >> OE_TERMINAL = "screen"
```
ref> https://forums.xilinx.com/t5/Embedded-Linux/petaconfig-c-kernel-error/td-p/764606

### Configure the kernel
```
$ petalinux-config -c kernel
```

```
Disable 'General setup > Initial RAM file system and RAM disk (initramfs/initrd) support'  
Disable 'Device Drivers > Hardware Monitoring support > PMBus support > Maxim MAX20751'  
Disable 'Bus Support > PCI support'  
Enable 'Kernel hacking > Tracers > Kernel Function Tracer'  
```

Save and Exit the kernel configuration

### Configure the rootfs
- packages
```
$ petalinux-config -c rootfs
```
```
Filesystem Packages --->
    misc --->
        [*] packagegroup-petalinux-self-hosted
```
- device tree
```
$ vi /tools/Xilinx/PetaLinux/2018.3/zcu102-petalinuxbase/project-spec/meta-user/recipes-bsp/device-tree/files/system-user.dtsi
```
```
  /include/ "system-conf.dtsi"                 
  / {
      cdmatest_0: cdmatest@0 {
          compatible = "xilinx,cdma-driver";
          dmas = <&axi_cdma_0 0>;
          dma-names = "cdma";
          dma-coherent;
      };
  };
```

### Creating bootable linux image
```
$ petalinux-build
$ petalinux-package --boot --fsbl images/linux/zynqmp_fsbl.elf --u-boot images/linux/u-boot.elf --pmufw images/linux/pmufw.elf --atf images/linux/bl31.elf --fpga images/linux/system.bit --force
or
$ petalinux-package --boot --fsbl images/linux/zynqmp_fsbl.elf --u-boot images/linux/u-boot.elf --force

$ cp images/linux/BOOT.BIN ~/build_results
$ cp images/linux/image.ub ~/build_results
$ cp images/linux/rootfs.tar.gz ~/build_results
```
Go to `Copy to SD card`

```
$ petalinux-build -x mrproper
```

### Another method to use Petalinux
Using the prebuilt image from `https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18842112/Zynq+UltraScale+MPSoC+Ubuntu+part+1+-+Running+the+Pre-Built+Ubuntu+Image+and+Power+Advantage+Tool`.</br>
Download `https://www.xilinx.com/member/forms/download/design-license-xef.html?filename=zynqus_pwr_zcu102_20171220.zip&akdm=1`.</br>
```
$ unzip zynqus_pwr_zcu102_20171220.zip
$ cd zynqus/pwr/sd
$ unzip 20171214_zcu102_ubuntu.zip
>> 20171214_zcu102_ubuntu.img
```
- `20171214_zcu102_ubuntu.img` is for `rootfs`
```
$ sudo dd if=20171214_zcu102_ubuntu.img of=/dev/sdc2
```

### SD card
1. #### Generate SD card partition
```
$ sudo fdisk -l
$ sudo fdisk /dev/sdc
> p

partition 1
> n > p > enter > enter > +1G
> a > p

partition 2
> n > p > enter > enter > enter
> p > w

partition 1
> m
> t > 1 > L > c

partition 2
> t > 2 > L > 83
> w
```
You must do partition formatting

2. #### Partition formatting
```
$ sudo mkfs.vfat -F 32 -n boot /dev/sdc1
(> mount시 권한이 peta:peta로 됨)
$ sudo mkfs.ext4 -L root /dev/sdc2

Disk /dev/sdc: 59.5 GiB, 63864569856 bytes, 124735488 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x99a003b9

Device     Boot   Start       End   Sectors   Size Id Type
/dev/sdc1          2048   1885973   1883926 919.9M  c W95 FAT32 (LBA)
/dev/sdc2       1886208 124735487 122849280  58.6G 83 Linux
```

### Copy to SD card
- #### How to mount
```
$ mkdir ~/mnt_dir
$ sudo fdisk -l
$ sudo mount /dev/sdc1 ~/mnt_dir
$ sudo umount ~/mnt_dir
$ df -h
$ du -hd1
$ sudo eject /dev/sdc
```

- #### Copy BOOT.BIN and image.ub to `boot` directory
Copy `BOOT.BIN`, `image.ub` by `cp` on partition 1, `boot` directory.
```
$ mkdir ~/mnt_dir
$ sudo fdisk -l
$ sudo mount /dev/sdc1 ~/mnt_dir
$ cp /tools/Xilinx/PetaLinux/2018.3/zcu102-petalinuxbase/images/linux/BOOT.BIN ~/mnt_dir
$ cp /tools/Xilinx/PetaLinux/2018.3/zcu102-petalinuxbase/images/linux/image.ub ~/mnt_dir
$ sudo umount ~/mnt_dir
```

- #### Copy the main directories to `root` directory
Copy the image of `rootfs.ext4` by `dd` on partition 2, `root` directory.
```
$ sudo dd if=rootfs.ext4 of=/dev/sdc2
$ sudo eject /dev/sdc
```

### Cross Compile
```
echo $PETALINUX
CC = /tools/Xilinx/PetaLinux/2018.3/tools/linux-i386/aarch64-none-elf/bin/aarch64-none-elf-gcc 
```


## Changelog

Detailed changes for each release are documented in the [release notes](https://github.com/l3l/ZCU102_petalinux/releases).

### Stay In Touch

- [Email]()

## Contribution

Please make sure to read the [Blog Guide]() before making a pull request. If you want to add more information, add it with a pull request to [this page](https://github.com/l3l/ZCU102_petalinux.git)!

Thank you to all the people who already contributed to ZCU102_petalinux!

ref>\
https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18841937/Zynq+UltraScale+MPSoC+Ubuntu+part+2+-+Building+and+Running+the+Ubuntu+Desktop+From+Sources

## License

[l3l]()

Copyright (c) 2021-present, l3l
