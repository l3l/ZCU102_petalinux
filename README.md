# ZCU102_petalinux

## Introduction
How to make petalinux on ZCU102 board

## Getting Started
###Install packages
```
$ sudo apt install -y python gawk gcc git make net-tools libncurses5-dev libncursesw5-dev zlib1g:i386 libssl-dev flex bison libselinux1 gnupg wget diffstat chrpath socat xterm autoconf libtool tar unzip texinfo zlib1g-dev gcc-multilib build-essential libsdl1.2-dev libglib2.0-dev screen pax gzip avrdude tftp tftpd-hpa tftp-server vim
$ sudo apt autoremove -y
$ sudo service tftpd-hpa status

$ sudo mkdir -p /tools/Xilinx/PetaLinux/2018.3
$ sudo chmod -R o+x /tools	
(dir: 775, file: 644)
$ sudo chown -R peta:peta /tools
$ cd /tools/Xilinx/PetaLinux/2018.3

$ mkdir ~/project_5_petalinuxBase
$ cp ~/Downloads/design_1_wrapper.hdf ~/project_5_petalinuxBase

$ mkdir ~/build_results

$ chmod +x ~/Downloads/petalinux-v2018.3-final-installer.run
$ ~/Downloads/petalinux-v2018.3-final-installer.run /tools/Xilinx/PetaLinux/2018.3
```

/************************* Configure the Petalinux project with the hardware description file *************************/
$ source settings.sh
$ petalinux-create --type project --template zynqMP --name ultra96-petalinuxbase
or
$ petalinux-create --type project --source ~/Downloads/xilinx-zcu102-zu9-es2-rev1.0-v2018.3-final.bsp --name zcu102-petalinuxbase
$ cd zcu102-petalinuxbase
$ petalinux-config --get-hw-description=~/project_5_petalinuxBase

Enable 'Image Packaging Configuration > Root file system type > SD card'
Disable 'DTG Settings > Kernel Bootargs > [] generate boot args automatically'
name
> zcu102-rev1.0
or
> avnet-ultra96-rev1
Set following kernel bootargs
> earlycon clk_ignore_unused earlyprintk root=/dev/mmcblk0p2 rw rootwait cma=1024M
or
> earlycon console=ttyPS0,115200 clk_ignore_unused root=/dev/mmcblk0p2 rw rootwait
or
> earlycon earlyprintk clk_ignore_unused root=/dev/mmcblk0p2 rw rootwait uio_pdrv_genirq.of_id=generic-uio cma=1024M rootfstype=ext3 init=/bin/sh stdout-path=serial0:115200n8

Enable 'Subsystem AUTO Hardware Settings > Serial Settings > Primary stdin/stdout ( ) > psu_uart_1'
Enable 'Subsystem AUTO Hardware Settings > Memory Settings > System memory size > 0x6FFFFFFF'
Enable 'Subsystem AUTO Hardware Settings > SD/SDIO Settings > Primary SD/SDIO ( ) > psu_sd_0'
Enable 'ARM Trusted Firmware Compilation Configuration > ATF memory settings'


Save and Exit the Petalinux project configuration

$ petalinux-build -c bootloader -x distclean


## Changelog

Detailed changes for each release are documented in the [release notes](https://github.com/l3l/github/releases).

### Stay In Touch

- [Email]()

## Contribution

Please make sure to read the [Blog Guide](https://git-scm.com/book/ko/v2/Git-브랜치-리모트-브랜치#_delete_branches) before making a pull request. If you want to add more information, add it with a pull request to [this page](https://github.com/l3l/github.git)!

Thank you to all the people who already contributed to github!


## License

[l3l]()

Copyright (c) 2021-present, l3l
