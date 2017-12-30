# LINUX KERNEL COMPILE ON QEMU INTEGRATOR/CP BOARD ON WSL
## Environment
* LINUX KERNEL 4.1.47 lowest LTS on version 4
* WSL (Ubuntu 16.04. Kernel ver. 4.4.0-43)
* QEMU 2.5.0
* ARM-NONE-EABI-GCC 6-2017-q2-update 6.3.1

## Linux menuconfig
### Set compile environment
    $ cd (linux kernal directory)
    $ export ARCH=arm
    $ export CROSS_COMPLE=arm-linux-gnueabi-
    $ make integrator_defconfig (at first time) or $ make menuconfig (from second time)

### CONFIG_PRINTK_TIME
    Kernel hacking --->
    printk and dmesg options --->
    [*] Show timing information on printks

### CONFIG_DEBUG_LL
    Kernel hacking --->
    [*] Kernel low-level debugging functions (read help!)

### CONFIG_DEBUG_LL_UART_PL01X (CONFIG_DEBUG_LL)
    Kernel hacking --->
        Kernel low-level debugging port --->
        Kernel low-level debugging via ARM Ltd PL01x Primecell
        (0x16000000) Physical base address of debug UART
        (0xf1600000) Virtual base address of debug UART

### CONFIG_EARLY_PRINTK (CONFIG_DEBUG_LL)
    Kernel hacking --->
    [*] Early printk

## zImage 컴파일
    $ make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- -j2 zImage

## QEMU 실행
    $ qemu-system-arm -machine integratorcp -kernel ./arch/arm/boot/zImage -m 256M
QEMU에서 Ctrl+Alt+3 누르면 serial0 console로 전환가능<br>
혹은 아래 명령으로 QEMU 실행시 serial 출력을 stdio에서 볼 수 있음

    $qemu-system-arm -machine integratorcp -kernel ./arch/arm/boot/zImage -m 256M -serial stdio

QEMU 실행 후 serial0 console에서 다음과 같이 출력되면서 중지됨

>Uncompressing Linux... done, booting the kernel.<br>
>no ATAGS support: can't continue
ATAGS 옵션 설정이 필요할 듯?

### CONFIG_ATAGS
    $ make menuconfig
<br>

    Boot options --->
    [*]   Support for the tradditional ATAGS boot data passing

## zImage compile 후 QEMU 실행
아래와 같이 에러발생. <br>
Device Tree를 입력해주어야 하는데 입력해주지 않아 발생한 에러

>Error: unrecognized/unsupported machine ID (r1 = 0x00000113).
>
>Available machine support:
>
>ID (hex)        NAME<br>
>ffffffff        Generic DT based system<br>
>ffffffff        ARM Integrator/AP (Device Tree)<br>
>ffffffff        ARM Integrator/CP (Device Tree)<br>
>
>Please check your kernel config and/or bootloader.

## DEVICE TREE 입력
### DEVICE TREE FILE 생성 (DTB FILE) : KERNEL COMPILE
    $ make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- -j2

### QEMU 실행시 DTB옵션 설정
    $ qemu-system-arm -machine integratorcp -kernel ./arch/arm/boot/zImage -m 256M -serial stdio -dtb ./arch/arm/boot/dts/integratorcp.dtb

QEMU 실행 후 아래 메시지를 출력하면서 커널패닉.<br>
root file system이 없다면서 종료됨.<br>
root file system만 생성해 주면 부팅 될 듯?

> end Kernel panic - not syncing: VFS: Unable to mount root fs on unknown-block(1,0)

## ROOT FILESYSTEM 만들기

여기서부터는 아래 사이트 참조.<br>
https://medicineyeh.wordpress.com/2016/03/29/buildup-your-arm-image-for-qemu/

### FHANDLE 적용
    General setup  --->
    [*] open by fhandle syscalls

### devtmpfs 적용
    Device Drivers  --->
    Generic Driver Options  --->
    [*] Maintain a devtmpfs filesystem to mount at /dev
    [*]   Automount devtmpfs at /dev, after the kernel mounted the rootfs

### BUILDROOT 적용 (X)
root file system을 만들기 위해 buildroot 사용.<br>
WSL에서 문제발생... 컴파일러 복사 에러

### CPIO로 ROOT FILESYSTEM 만들기
cpio 유틸로 파일시스템 생성<br>
SYSV, V7, ROMFS 파일시스템 지원 추가
    
    $ echo final_image.elf | cpio -o --format=newc > rootfs

qemu 실행할 때 initrd 옵션 추가

    qemu-system-arm -machine integratorcp -kernel ./arch/arm/boot/zImage -m 256M -dtb ./arch/arm/boot/dts/integratorcp.dtb -serial stdio -initrd ../my_os/rootfs

아래 메시지를 출력하면서 커널패닉
>
>[    1.222003] No filesystem could mount root, tried:  ext2 cramfs vfat sysv v7 romfs
>
생성된 rootfs가 현재 컴파일된 zImage에서 지원되지 않음<br>
어떤게 지원이 되고 어떤게 지원이 안되는지?<br>
모든 파일시스템 지원을 포함하면 되는지?

아니고, rootfs가 실행되는 명령을 추가했을 경우 hang 됨.<br>
    
    -append "root=/dev/ram rdinit=main"


## Compiler 변경
컴파일러가 문제다....<br>
arm-none-eabi-gcc 로 컴파일 했을 때는 제대로된 실행파일이 생성되지 않음.<br>
윈도우 버전의 컴파일러라 그랬을 수 있나??? 커널은 이걸로 컴파일 했는데???

arm-linux-gnueabi-gcc 로 컴파일 후에 <br>
https://landley.net/writing/rootfs-howto.html <br>
참조하여 initramfs 만든 후 실행하니 깔끔하게 잘 됨. <br>

## CLDC 화면에 출력
화면은 왜 안나올까???

    Device Drivers --->
      Graphics support --->
        Frame buffer Devices --->
          <*> Support for frame buffer devices
          <*> ARM PrimeCell PL110 support

        Console display driver support --->
          <*> Framebuffer Console support

        [*] Bootup logo --->
    
설정 후 커널컴파일 후 QEMU 재실행 하니 로고가 나옴.. <br>
성공!!<br>
아래명령으로 실행하면 console에서 리눅스 부팅메시지 확인가능

    $ qemu-system-arm -machine integratorcp -kernel ./arch/arm/boot/zImage -m 256M -dtb ./arch/arm/boot/dts/integratorcp.dtb -serial stdio -initrd ../my_os/initramfs_data.cpio.gz -append "console=tty0 earlyprintk init=/init"

완전 성공!!!
