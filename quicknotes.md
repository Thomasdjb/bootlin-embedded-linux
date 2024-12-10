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


