## Day 1

### Morning

MMU ? memory management unit  
hard to run linux wo MMU  
linux can run on 8mb of ram but more realistically 32mb  
storage > 4mb  
MMC -> multimedia card  
SoM -> system on module = chip + memory + connector to access  other peripherals  
when developing a new system, try to find a chip that is supported by the mainline linux kernel  
cross compilation toolchain to compile on the host machine
execute bootloader from sRam  
bootloader initialize RAM  
kernel handle memory management, filesystems  
BSP : board support package  
contains device drivers for targeted hw  
libc isnt part of the kernel  
bootROM little code in ROM that load the bootloader  
often 2 steps bootloaders  
binutils -> set of tools to manipulate binaries (usually .elf) -> GPL license  
strip -> to remove part that are needed for debugging  
libc connect kernel to user space applications  
need kernel header to compile libc  
kernel header contains cst, define, struct linked to user operations (read/write operation number, ...)  
every version of the kernel need to be retrocompatible with older kernels   
usually everyone use glibc or uClibc/musl if we have memory constraints  
MIT can link with static applications without need to divulgue software  
GPL must share application code  
LGPL is in between MIT and GPL  
ABI -> Application binary interface   
defines how instruction to machine code should be written (arm, risc, ..)  
floating point support: arm7 and risc-v have FPU, toolochain   needs to decide which ABI to use if the hw support FPU  
gnu tools need to be compiled for a specific architecture  
building a toolchain manually: hard process  
other way, download a pre compiled toolchain  
other way use utilities that automate the building process -> crosstool-ng  
other way is to use a tool that build the toolchain and the filesystem -> ex buildroot, yocto  

### Lab 1: compiling our own toolchain

add ./bin to path after toolchain compilation to have all the tools in the PATH

### Afternoon

bootloader does hw init, loading application, decompression of binaries app, execution of app  
most bootloader provides a shell: load data, tftp  
#### on x86
BIOS = basic input output system  
MBR = master boot program  
BIOS equivalent for embedded is ROMCode  
BIOS  
uefi = unified extensible firmware interface  
acpi = advanced configuration and power interface  

#### embedded
rom code: implement initial step of the ROM code  
written by processor vendor  
responsible to find suitable bootloader  
copy bootloader in sRam and execute it from there  
can configure boot using register configuration  

grub = grand unified bootloader  
standart on x86 platform  
systemd-boot = alternative for grub, not really used  
shim = signed by microsoft = can be used for secure boot  
u-boot = most used in embedded  
barebox = successor of u-boot  

arm = 4 priviledge level
- user space
- linux kernel
- hypervisors
- runs secure firmware

2 worlds
- normal world
- secure world (trustzone on arm)

linux kernel will do specific system call to the secure world to access secure peripherals for example  
armv8 mandates to load a secure firmware  
TF-A trusted firmware A: reference app of trusted firmware  
firmware without libc  

RISC-V processor also have security level 
3 level:
- M mode: machine mode
- S mode: linux kernel level
- U mode: user space applications

boot sequence:  
ROMCode -> U-boot SPL -> U-Boot -> linux kernel  
what is FAT ? fat is a filesystem format  


#### u-boot

try to find an upstream uboot for your soc  
configuration with kconfig  
configs/ directory  
some board have not be moved to kconfig  
defconfig vs .config : devconfig list only not default   configuration defines while .config list all  
u-boot must be configured before being compiled  
cross compiler must be configured through CROSS_COMPILE variable  
SPL = secondary program loader  
SPL is first stage of the bootloader    
will load the bootloader into the sRAM    
SPL bootloader behavior is hardcoded in C  

device tree = data structure that describe topology of hardware  
u-boot has environment variable: key/value pairs  
loaded and modified in RAM  
environment can be saved in nvm  
ext4 is the default filesystem of many linux distributions  
uboot is loaded at the end of the RAM  
uboot commands used to manage memory use RAM address as input  
uboot supports many filesystem (FAT, ext2,3,4 , squashfs)  
uboot doesnt answer ping  
uboot environment commands can contains small script  
if then else fi syntax exist  
FIT = flat image tree, more used nowadays  
contains multiple kernel image, device trees, initramfs, ...  
ensure binaries doesn't overlap in memory
.its file describe content of the image
.itb after compilation by the Device Tree Compiler
generic distro boot: file that specifies uboot how to boot  
can be specified as an arg of uboot bootcmd  

TF-A specific concept FIP: firmware image package  
TFA does not use kconfig  
everything comes from the make command line  
TF-A important variable:
- CROSS_COMPILE
- ARCH
- ARM_ARCH_MAJOR
- PLAT
- AARCH32_SP
- DTB_FILE_NAME
- BL33


#### TP 1 

- compile crosstool ng to be able to compile a cross compilation toolchain
- compile a cross compilation toolchain for ARM stm32mp1
- compile tf-a for the target
- erase the sd card
- partition the sd into 4 partitions: 2* same partition / FIP / bootfs
- format the boot partition (4) to ext4 filesystem format
- write tfa binary on partition 1,2
- write the fip partition with the FIP which contains U-boot, BL32 monitor and device tree
- load the sd card on the target and boot
- you should be able to access the uboot shell with a serial link



## Day 2

### Morning

#### Linux kernel

Role is to manage hw resources: cpu, memory, io  
provide set of portable APIs to interface with user app  
handle concurrent access  
system and kernel info accesible through pseudo filesystems:
- proc, /proc, operating system related information
- sysfs, /sys, representation of system a tree of device connected by buses  

we try to select a latest kernel and try to forward port the missing drivers  
linux kernel code size is 60% drivers / 12% arch  
device trees have an include component  
arm32: use bootz with a zImage file  
arm64 and RISCV: booti with Image file  
some kernel configuration can be adjusted on command-line  
rootwait to have the kernel wait before booting and avoid Kernel panic  


#### TP 2

- configure the linux kernel
- compile the kernel for the target
- upload the zImage and .dtb to the board via tftp
- boot on the zImage
- automate booting using tftp and `bootcmd` variable to create a script

setenv bootargs ${bootargs} root=/dev/nfs ip=192.168.0.114 nfsroot=192.168.0.1:/home/fomation-07/work/nfsroot,nfsvers=3,tcp rw

#### Afternoon

##### filesystems

`mount` without argument list all mount points on the system  
/ is the root filesystem
need to mount a storage device to access its filesystem
need to umount before unplugging  
root fs can be mount from partition of external memory  
its a config of the kernel behavior  
partition:
- usb or hard disk: /dev/sdXY where x is device number and y partition number
- sd card: /dev/mmcblkXpY where x is device number and y partition number
- flash storage: mtdblockX where x is partition number  

we can also mount the root filesystem from a network filesystem (NFS)  
we need to install an nfs server (`sudo apt install nfs-kernel-server`)   
and add the directory we want to share to the `/etc/exports` file  
we need to compile the kernel with the according commands to have NFS support  
we can also boot from filesystem in memory: `initramfs`  
this can be used as an intermediate step before switching to real fs or to have a very fast boot  
initramfs is a CPIO archive: `cpio -H newc -o > initramfs.cpio && gzip initramfs.cpio`
we can also do it at the compilation time with setting `CONFIG_INITRAMFS_SOURCE` to our root fs  
important directory of root fs:
- bin: basic programs
- boot: kernel image, config and initramfs
- dev: device files
- etc: system wide config
- home: user directory
- lib: basic libs
- media: mount point for removable medias
- mnt: mount point for temporarily mounted filesystem
- proc: mount point for the proc virtual filesystem
- root: home directory of root user
- run: run time variable data
- sbin: basic system programs
- sys: mount point of the sysfs virtual fs
- tmp: temporary files
- usr: user specific bin, sbin, lib
- var: variable data files for system services. temporary file, logging

##### pseudo filesystems

proc: kernel expose statistics about running processes in the system  
ps and top use `/proc` fs to print data  
command to mount proc: `mount -t proc nodev /proc`  
proc contains one directory for each running process  
contains details about the files opened by the process, CPU and memory usage ...  
`/proc/sys` contains file that can be written to adjust kernel parameters
ex: can be used to drop the cache and see real memory usage of an application

##### minimal filesystem

minimal requirements:  
an init application: first user space application started after mount of the root fs  
init is responsible for starting all other user space apps and services  
is the original parent process  
shell is not mandatory but will always be present  
systemd can be used as replacement of the init application  
on embedded device, it can increase a lot the boot time compared to a simple init app  
it also take more processing and storage so it wont fit on every target  
kernel doesnt need specific module to read an initramfs  

#### BusyBox

is a rewrite of many useful unix command line utilities  
compiled into a single executable  
symbolic links between each command and `/bin/busybox`  
ARG0 is used to call the corresponding command  
busybox is used in debian and ubuntu to implement some commands  
busybox contains implementation of an init program  
busybox can tell how much an option will cost in memory  


#### TP 3

- compile busybox
- load busybox fs with nfs on the board
- create inittab and init.d/rCS script to init
- cross compile binary for target
- try to run on target -> wont work -> add static missing libraries -> run ex
- measure size of busybox binary before: 337.2K
- after compile without static: 218.1K
