# alexghiti.github.io

riscv XIP kernel
================

At the moment, it was only tested on Microchip Polarfire SoC.

Quick way
---------

1. Download SD card image from https://github.com/polarfire-soc/meta-polarfire-soc-yocto-bsp/releases

```
$ wget https://github.com/polarfire-soc/meta-polarfire-soc-yocto-bsp/releases/download/v2021.02/core-image-minimal-dev-icicle-kit-es-sd-20210216171231.rootfs.wic.gz
```

2. Hart Software Services

Follow instructions here https://github.com/polarfire-soc/hart-software-services#building (you can use [ .config](https://github.com/AlexGhiti/alexghiti.github.io/blob/main/xip/hss/.config) based on 0.99.15 as I had some issues building it).

3. Linux kernel

Build using the config here: [ .config ](https://github.com/AlexGhiti/alexghiti.github.io/blob/main/xip/kernel/.config)

4. Launching using qemu

Support for Polarfire in qemu is recent, more instructions can be found here: [qemu Polarfire](https://wiki.qemu.org/Documentation/Platforms/RISCV#Microchip_PolarFire_SoC_Icicle_Kit)

Use the following command line (adapting the path):

```
$ qemu-system-riscv64 												\
	-M microchip-icicle-kit											\
	-bios hart-software-services/Default/hss.bin								\
	-smp 5 -display none -nic user,model=cadence_gem -serial null						\
	-chardev socket,id=serial1,path=/tmp/serial1.sock,server -serial chardev:serial1			\
	-drive if=sd,format=raw,file=core-image-minimal-dev-icicle-kit-es-sd-20210216171231.rootfs.wic.gz	\
	-device loader,addr=0x21000000,file=arch/riscv/boot/xipImage						\
	-m 6G -D qemu.log -monitor stdio -s
```

Launch the following to connect to the serial output:

```
$ sudo minicom -D unix#/tmp/serial1.sock
```

In uboot console, launch the XIP kernel using the following command:

```
RISC-V # go 0x21000000
```

And in parallel, you can launch gdb using the following gdb commands:

```
(gdb) target extended-remote :1234
(gdb) add-inferior
(gdb) inferior 2
(gdb) attach 2
(gdb) set schedule-multiple
(gdb) info threads
```

Generally I don't have any output in minicom from the kernel (I have not found why yet). But I can see kernel messages by compiling the kernel with following config:

```
CONFIG_DEBUG_INFO=y
CONFIG_GDB_SCRIPTS=y
```

And by using the following command in gdb:

```
(gdb) file vmlinux
(gdb) lx-dmesg
```

Long way (never tested)
-----------------------

See https://github.com/polarfire-soc/polarfire-soc-buildroot-sdk
