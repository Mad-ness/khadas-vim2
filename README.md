# khadas-vim2
Some notes about using Khadas VIM2 board. The notes are oriented on Gentoo users. It's suggested that the bootloaded (u-boot) has not been overwritten.

To simplify some work, install any supported Linux OS like Armbian to run the board first. I used the image announced there http://forum.khadas.com/t/gentoo-vim2-stage3-vanilla-kernel/1607. And I had some problems with **dd** this image on my microSD card – my one has been a couple of hundreds bytes smaller than the downloaded image built for. If you have the same problem and still *dd* the image it will not load complaining on the damaged root filesystem. I did some tricks to decrease its size to 5GiB and it booted fine. This image also saved me much time on installing Gentoo because all what I did here was recompiling the packages on my hardware (by default it has all versions unmasked and very latest versions of packages are installed, I just need more stability), removing unneeded packages, installing needed ones.

Note: all commands are run in the /boot directory and all paths use this as a root if not stated otherwise.
Note: I don't like using initramfs (initrd), so all what you need to make the kernel and the board run should be compiled into the kernel such as eMMC support.
Note: I'm not an expert neither in Gentoo, nor in single board computers (SBC). Although I use Gentoo for over 8 years and 4+ years the SBCs. Many things I still don't understand.

## The /boot content

To properly run the board some things to be done in the /boot.

- script: **s905_autoboot.cmd**
- kernel image: **uImage**
- dtb file.

The **/boot** is a FAT32-formatted file system.

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
Run this command every time when you modify the script.

### Preparing a proper kernel image

Ok, compile the kernel with the options you need and install it, including modules and device tree blobs.

```shell
cd /usr/src/linux
make -j8 Image modules dtbs
# waiting for a long time if you compile it on the board itself
make modules_install
cp arch/arm64/boot/Image /boot/Image
cp arch/arm64/boot/dts/amlogic/*dtb /boot/dtb/
```
Other target like Image.gz I couldn't boot. Althought the u-boot provides the ***booti*** command for running compressed kernels, it didn't work for me like it works in the Armbian OS and the images built by the balbes150. I don't know why, the only difference is I don't have the Initrd image.


Ok, we've almost done.


I've got a working kernel only using these steps
```shell
mkimage -A arm64 -O linux -T kernel -a 0x1080000 -e 0x1080000 -C none -n uImage -d Image uImage
```
- the keypoint here is the ***0x1080000*** address. Only this value started working, I've found it there http://forum.khadas.com/t/compiling-the-new-firmware/1119/15.
- that command gives the **uImage** image that is the kernel wrapped for feeding the u-boot.

To verify the above command, run this:
```shell
file uImage
u-boot legacy uImage, uImage, Linux/ARM 64-bit, OS Kernel Image (Not compressed), 18317824 bytes, Tue Jun 12 07:49:30 2018, Load Address: 0x01080000, Entry Point: 0x01080000, Header CRC: 0x28F167CD, Data CRC: 0x56B44986
```

And yes, the kernel is very thick now. The config for this has been taken from the latest Armbian, 4.16.3 version and adopted for 4.17 (***make oldconfig***). There is still too much work to make it light.

### Summarize
Probably that's all. You're responsible for installing and configuring everything else.

### Points to improve
- as far as I understand u-boot can open files from ext2/3/4 file systems.
