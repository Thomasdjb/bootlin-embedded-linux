# Day 1

## Morning

MMU ? memory management unit  
hard to run linux wo MMU  
linux can run on 8mb of ram but more realistically 32mb  
storage > 4mb  
MMC -> multimedia card  
SoM -> system on module = chip + memory + connector to access other peripherals  
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

## Afternoon

bootloader does hw init, loading application, decompression of binaries app, execution of app  
most bootloader provides a shell: load data, tftp  

### on x86

BIOS = basic input output system  
MBR = master boot program  
BIOS equivalent for embedded is ROMCode  
BIOS  
uefi = unified extensible firmware interface  
acpi = advanced configuration and power interface  

### embedded

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


### u-boot

try to find an upstream uboot for your soc  
configuration with kconfig  
configs/ directory  
some board have not be moved to kconfig  
defconfig vs .config : devconfig list only not default  
configuration defines while .config list all  
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


### TP 1

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



# Day 2

## Morning

### Linux kernel

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

### TP 2

- configure the linux kernel
- compile the kernel for the target
- upload the zImage and .dtb to the board via tftp
- boot on the zImage
- automate booting using tftp and `bootcmd` variable to create a script
  

`setenv bootargs console=ttySTM0,115200 root=/dev/nfs ip=192.168.0.114 nfsroot=192.168.1.115:/home/fomation-07/work/nfsroot,nfsvers=3,tcp rw`  

## Afternoon

### filesystems

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

### pseudo filesystems

proc: kernel expose statistics about running processes in the system  
ps and top use `/proc` fs to print data  
command to mount proc: `mount -t proc nodev /proc`  
proc contains one directory for each running process  
contains details about the files opened by the process, CPU and memory usage ...  
`/proc/sys` contains file that can be written to adjust kernel parameters  
ex: can be used to drop the cache and see real memory usage of an application  

### minimal filesystem

minimal requirements:  
an init application: first user space application started after mount of the root fs  
init is responsible for starting all other user space apps and services  
is the original parent process  
shell is not mandatory but will always be present  
systemd can be used as replacement of the init application  
on embedded device, it can increase a lot the boot time compared to a simple init app  
it also take more processing and storage so it wont fit on every target  
kernel doesnt need specific module to read an initramfs  

### BusyBox

is a rewrite of many useful unix command line utilities  
compiled into a single executable  
symbolic links between each command and `/bin/busybox`  
ARG0 is used to call the corresponding command  
busybox is used in debian and ubuntu to implement some commands  
busybox contains implementation of an init program  
busybox can tell how much an option will cost in memory  


### TP 3

- compile busybox
- load busybox fs with nfs on the board
- create inittab and init.d/rCS script to init
- cross compile binary for target
- try to run on target -> wont work -> add static missing libraries -> run ex
- measure size of busybox binary before: 337.2K
- after compile without static: 218.1K


# Day 3

## Morning

### Hardware access

bus controller driver: i2c, spi, usb, pci  
bus subsytem provides API for drivers to access a type of bus  
device driver: drives a particular device to a given bus  
driver subsystem expose feature a device through standard kernel/user-space interface  
application can access device through this  
kernel driver expose a standard API to user-space application  
hide particularity of a specific device, user just need to know standard device type interface  
it possible to access device directly from user space application  
useful for very specific devices, not recommended  
High level, 3 interfaces to access a device exposed by the kernel:
- device nodes in /dev
- entries in the sysfs filesystem
- network sockets and related APIs  

devices in /dev:  
  
2 differents abstraction:
- block device
- character device  

internally, the kernel identifies a device by 3 informations:
- type (character or block)
- major (category of device)
- minor (id of device)  

block device: device composed of fixed-sized blocks: sd cards, hard disk, usb keys  
character devices: infinite stream of bytes, no beginning, no end, no size: a serial port, sound card, video acquisition, frame buffers  
unix, most system objects as files  
application can manipulate system object with normal file API (open,read,write,close,...)  
device are represented through a device file to applications  
device file: special type of file that associate file name to the triplet that kernel knows  

#### devtmpfs

devtmpfs virtual filesystem can be mounted on `/dev` so the kernel automatically creates/remove device files  
devtmpfs is great but capabilities limited so complementary solutions exist:  
udev: daemon that receive event of the kernel about devices  
can create remove device files, adjust permission, create symbolic links to devices  
almost in all linux distribution, ex: `/dev/by-unid/`  
mdev: lightweight implementation of udev, part of busybox  

#### sysfs

sysfs filesystem:
- `block/`: symlink to all block devices in /sys/devices
- `bus/`: one sub folder by bus
- `class/`: one sub folder by class: input, leds, pwm
- `dev/`: block/ one symlink per block device; char/ one symlink per character device
- `devices/`: all in the system
- `firmware/`: representation of firmware data
- `fs/`: filesystem drivers
- `kernel/`: properties of kernel subsystems
- `module/`: properties kernel module
- `power/`: power management related properties
  
`/dev/` vs `/sysfs/` :  
`/dev` to access the device  
̀`/sysfs` properties of the devices  
some device only have a sysfs interface  

#### GPIO

legacy interface /sys/class/gpios  
new interface libgpiod: /dev/gpiochipx  
need to compile C lib  

#### other virtual fs

debugfs: `/sys/kernel/debug`, contains debug info from kernel  
configfs: `/sys/kernel/config`, manage config of advanced kernel mechanism  

#### kernel modules

kernel module to keep the kernel image minimal  
installed in `/lib/modules/<kernel-version>/`  
stored in `.ko` (kernel object) files  
kernel modules can depends on other modules  
dependencies described in `modules.dep`  
`modinfo` for information about modules  
`lsmod` to list loaded kernel modules  
`insmod` to load a kernel module  
`rmmod` to remove a kernel module  
`modprobe` advanced tool to load modules  

#### device trees

some embedded device are not discoverable  
need to write a device tree for our embedded target  
`.dts`: device tree source, describing the HW  
`dtc`: device tree compiler  
`.dtb`: device tree blob produce by compiler, also called FTD, flattened device tree  
device to store the dts are discussed but they are often cloned between the projects (kernel, u-boot, tf-a)  
syntax: 
- tree of nodes
- each node has properties
- nodes is ~ a device or IP block
- properties are ~ device characteristics
- notion of celles in property values
- notion of phandle to point to other node
  
dtc does not do semantic validation, only syntax  
device tree can be split in several files, and include each others  
`.dtsi` are included files while `.dts` are final device trees  
`.dtsi` will include SoC level information  
`.dts` contains board level information  
dts is used to describe the HW not to configure  
describe integration of HW component not internal HW components  
device tree specifications exist  
device tree bindings -> describe how a piece of HW should be described  
device tree bindings are written in YAML and provides semantic validation  
kernel provides `make` rules to check device tree  

##### compatible property

list of string, allows kernel to find appropriate driver for this device  
from most specific to least specific  
in form `vendor, model`  
special value `simple-bus` where all sub nodes are memory mapped devices  
linux identifies as platform devices: top level DT nodes with compatible string or sub nodes of `simple-bus`  

##### reg property

memory mapped devices: base physical address and size of the memory mapped registers  
I2C: address of the device on the I2C bus  
SPI: chip select number  
unit address must be adress of the first reg entry  

##### status property

indicates if device is used or not  
`okay` or `ok`: device is in use  
any other value is disabled, device is not in use  

##### pin muxing

need to mux pins to expose all SoC functionnalities to the oustide world  
dts describes muxing  

### TP 4: hardware access

#### GPIO IN

- config kernel to enable legacy interface (/sys/dev/gpio)
- build kernel
- boot with need kernel
- mount debugfs to /sys/kernel/debug/
- connect PE1 to GND
- PE1 is part of GPIO E bank -> correspond to `gpiochip4`
- PE1 is second pin on this bank and bank is 576 to 591 so PE1 is 577
- `echo 577 > export`: expose a PE1 device file in `/sys/class/gpio/`
- `echo in > PE1/direction`: set the in direction for PE1
- `cat PE1/value`

#### LED

- go to `/sys/class/leds`
- go to led `heartbeat`
- you can configure the trigger from multiple source, here is heartbeat so its CPU activity
- disable all trigger `echo none > trigger`
- control the led directly with `echo 1 > brightness`
- you can control the led with a timer
- `echo timer > trigger`
- `echo 10 > delay_on`
- `echo 10 > delay_off`

## Afternoon

### Block devices

block devices can be read and written on a per block basis: ex: hard disk, ram disk, usb keys, ssd, sd cards, eMMC, ...  
raw flash devices are driven by a controller on the SoC: ex: NOR flash, NAND flash  
`proc/partitions/` to list all blocks devices  
block devices can be partionned to store different parts of the system  
tools to create partition of a block device: fdisk, cfdisk, sfdisk, parted  
block device in /dev/ allow raw acccess to the device, bypassing any filesystem layer  
`dd` is tool for theses transfers  

### Available block filesystems

ext2: earliest, still supported, risk of metadata corruption, no journal, not recommended today  
ext3: successor of ext2, add journal  
journal: write in the journal before writtin anything so we can recover if crash  
ext4: actively developped, default filesystem for many gnu/linux distribs  
xfs: journaling fs, since 1994, maintained by red hat  
btrfs: copy on write filesystem, modern, on desktops  
f2fs: log structured fs, started by samsung, designed for solid state based storage  
squashfs: read only and compressed fs, actively maintained, suitable for very small partitions, supports compression  
erofs: enhanced rad only fs, used in particular android phone (huawei, xiaomi, oppo, ...), too recent  
advice for chosing best fs: depends on partition size, speed, media type  
ext4 is the default choice  

#### compatibility filesystems

- vfat: compatibility with FAT fs used in the windows world
- ntfs: compatibility with windows NTFS fs
- hfs: compatibility with macos HFS fs

#### filesystem in RAM

tmpfs: fs in RAM, useful for temporary storage, more space efficient than ramdisk  
how to use ? `mount -t tmpfs run /run`  

#### using block filesystems

`mkfs.ext4 /dev/sda3` to create an empty ext4 fs  
create a filesystem from directory: `genext2fs -d rootfs/ rootfs.img`  
once created, you can access the fs from dev workstation  
`mkdir /mnt/test  && mount -t ext4 -o loop rootfs.img /mnt/test`  
you may also have a complete image with multiple partitions  
`sudo losetup -f --show --partscan disk.img sudo losetup -f --show --partscan disk.img`  
to create a directory per partition  
you can split a block device between read only partition and a read write partition  

### Flash storage and filesystem

MTD: memory technology devices  
generic subsytem dealing with all types of storage media  
supports RAM,ROM,NAND,NOR,Dataflash,...  
MTD devices are usually partionned  
partition of MTD device partition is described externally  
each partition is a new MTD device  
specific fs have been develop to deal with flash constraints  
these fs rely on MTD layer to access flash chips  
UBI/UBIFS standard for medium capacity NANDs  

### UBI

Unsorted block image  
standard file API -> ubifs -> ubi -> mtd driver -> flash  
UBI use Logical Erase Block to distribute evenly erases  
if you need partition, use UBI volumes, not MTD partitions  
exceptions: bootloaders will need to be MTD partitions  
try to group MTD partition at the beginning of the flash device  

### TP-4

# Day 4

## Morning

### Cross compiling libraries

build system: makefile, cmake, meson  
when cross compiling libraries, 2 copies of root fs:  
- target root fs contains only whats needed for runtime
- staging space contains header, static libs, doc, binaries with debug symbols  

#### make

make arg:
- CC: C compiler path
- CXX: C++ compiler path
- LD: linker path
- CFLAGS: C compiler flags
- CXXFLAGS: C++ compiler flags
- LDFLAGS: linker flags
- DESTDIR: for installation destination
- PREFIX: execution location  

#### autotools

autoconf handle configuration of software package  
automake generate makefile needed to build  
libtool handle generation of shared libs  
old, try not to use  
4 steps:
- `./configure`: configuration
- `make`: build
- `make install`: installation  

#### cmake

maintained by kitware  
losing traction these day for meson  
need cmake installed  
typical sequence:
- cmake .
- make
- make install  
to avoid long command lines with cmake you can use toolchain-file.txt  
`cmake -DCMAKE-TOOLCHAIN-FILE=/path/to/toolchain-file.txt`

#### meson

most modern, written in python  
processes `meson.build + meson_options.txt`  
`Ninja` is an alternative to make with shorter build time  
chain:
- `mkdir build`
- `cd build`
- `meson ..`
- `ninja`
- `ninja install`

#### pkg-config

tool to query database to get information on how to compile programs that depends on lib  


#### TP-5

`setenv bootargs console=ttySTM0,115200 root=/dev/nfs ip=192.168.0.114 nfsroot=192.168.1.115:/home/fomation-07/work/third-party/target,nfsvers=3,tcp rw`

