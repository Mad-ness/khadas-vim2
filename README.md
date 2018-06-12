# khadas-vim2
Some notes about using Khadas VIM2 board. The notes are oriented on Gentoo users. It's suggested that the bootloaded (u-boot) has not been overwritten.

To simplify some work install any Linux OS to run the board first.

Note:: all commands are run in the /boot directory and all paths use this as a root if not stated otherwise.

## The /boot content

To properly run the board some things to be done in the /boot.

- script: **s905_autoboot.cmd**
- kernel image: **uImage**
- dtb file.

### Script s905_autoboot.cmd

This script is needed to get the compiled **s905_autoboot** script.

```shell
setenv kernel_loadaddr "0x11000000"
setenv dtb_mem_addr "0x1000000"
setenv dtb_image "dtb/meson-gxm-q200.dtb"
setenv condev "console=ttyAML0,115200n8 console=tty0 no_console_suspend consoleblank=0"
setenv boot_start booti ${kernel_loadaddr} - ${dtb_mem_addr}
setenv bootargs "root=/dev/mmcblk1p5 rootflags=data=writeback rw ${condev} fsck.repair=yes net.ifnames=0 mac=${mac}"

fatload mmc 0 ${dtb_mem_addr} ${dtb_image}
fatload mmc 0 ${kernel_loadaddr} uImage
bootm ${kernel_loadaddr} - ${dtb_mem_addr}
```

To get the **s905_autoboot** script run the following command (the u-boot-tools atom (package) must be installed):
```shell
mkimage -C none -A arm64 -T script -d s905_autoscript.cmd s905_autoscript
```

