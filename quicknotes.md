## Day 1

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
floating point support: arm7 and risc-v have FPU, toolochain needs to decide which ABI to use if the hw support FPU
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
on x86
    BIOS = basic input output system
    MBR = master boot program
    BIOS equivalent for embedded is ROMCode
    BIOS
    uefi = unified extensible firmware interface
    acpi = advanced configuration and power interface
embedded
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
-user space
-linux kernel
-hypervisors
-runs secure firmware

2 worlds
-normal world
-secure world (trustzone on arm)

linux kernel will do specific system call to the secure world to access secure peripherals for example
armv8 mandates to load a secure firmware
TF-A trusted firmware A: reference app of trusted firmware
firmware without libc
