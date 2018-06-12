# khadas-vim2
Some notes about using Khadas VIM2 board. The notes are oriented on Gentoo users. It's suggested that the bootloaded (u-boot) has not been overwritten.

To simplify some work, install any supported Linux OS like Armbian to run the board first.

Note: all commands are run in the /boot directory and all paths use this as a root if not stated otherwise.
Note: I don't like using initramfs (initrd), so all what you need to make the kernel and the board run should be compiled into the kernel such as eMMC support. I used the image announced there http://forum.khadas.com/t/gentoo-vim2-stage3-vanilla-kernel/1607. And I had some problems with **dd** this image on my microSD card – my one has been a couple of hundreds bytes smaller than the downloaded image built for. If you have the same problem and still *dd* the image it will not load complaining on the damaged root filesystem. I did some tricks to decrease its size to 5GiB and it booted fine.

## The /boot content

To properly run the board some things to be done in the /boot.

- script: **s905_autoboot.cmd**
- kernel image: **uImage**
- dtb file.

### Script s905_autoboot.cmd

This script is needed to get the compiled **s905_autoboot** script.

The content of the s905_autoboot.cmd should be like this
```shell
setenv kernel_loadaddr "0x11000000"
setenv dtb_mem_addr "0x1000000"
setenv dtb_image "dtb/meson-gxm-q200.dtb"
setenv condev "console=ttyAML0,115200n8 console=tty0 no_console_suspend consoleblank=0"
setenv bootargs "root=/dev/mmcblk1p5 rootflags=data=writeback rw ${condev} fsck.repair=yes net.ifnames=0 mac=${mac}"

fatload mmc 0 ${dtb_mem_addr} ${dtb_image}
fatload mmc 0 ${kernel_loadaddr} uImage
bootm ${kernel_loadaddr} - ${dtb_mem_addr}
```

Importants notes:
- the addresses must be as specified in the script. Although the ***${dtb_mem_addr}*** can be omitted since it's defined before the script executes.
- ***/dev/mmcblk1p5*** points at the internal eMMC where the partition 5 is the root filesystem. ***mmcblk1*** is the internal memory (your sd card identified as ***mmcblk0***).
- the ***minus (-)*** in the bootm command says to not use Initrd image

To get the **s905_autoboot** script run the following command (the u-boot-tools atom (package) must be installed):
```shell
mkimage -C none -A arm64 -T script -d s905_autoscript.cmd s905_autoscript
```

