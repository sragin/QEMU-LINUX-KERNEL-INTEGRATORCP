#h1 리눅스 커널 설정
Type below on command prompt
    $make ARCH=arm menuconfig

printk 설정
 - Kernel hacking --> printk and dmesg options --> Show timing information on printks
CONFIG_DEBUG_LL
 - Kernel hacking -> Kernel low-level debugging functions
CONFIG_DEBUG_LL_UART_PL01X (CONFIG_DEBUG_LL)
                  -> Kernel low-level debugging port -> Kernel low-level debugging via ARM Ltd PL01x Primecell
CONFIG_EARLY_PRINTK (CONFIG_DEBUG_LL)
 - Kernel hacking -> Early printk


zImage 컴파일
make ARCH=arm CROSS_COMPILE=arm-none-eabi- -j2 zImage


QEMU 실행
qemu-system-arm -machine integratorcp -kernel ./arch/arm/boot/zImage -m 256M

QEMU에서 Ctrl+Alt+3 누르면 serial0 console로 전환가능
serial0 console에서 다음과 같이 출력되면서 중지됨
"
Uncompressing Linux... done, booting the kernel.
no ATAGS support: can't continue
"
ATAGS 옵션 설정이 필요할 듯

CONFIG_ATAGS
Boot options -> Support for the tradditional ATAGS boot data passing


다시 zImage 컴파일 후 QMEMU 실행
에러발생
"
Error: unrecognized/unsupported machine ID (r1 = 0x00000113).

Available machine support:

ID (hex)        NAME
ffffffff        Generic DT based system
ffffffff        ARM Integrator/AP (Device Tree)
ffffffff        ARM Integrator/CP (Device Tree)

Please check your kernel config and/or bootloader.
"

커널 컴파일 (DTB파일 생성)
make ARCH=arm CROSS_COMPILE=arm-none-eabi- -j2


QEMU 실행시 DTB옵션 설정
qemu-system-arm -machine integratorcp -kernel ./arch/arm/boot/zImage -m 256M -dtb ./arch/arm/boot/dts/integratorcp.dtb -serial stdio
"
---[ end Kernel panic - not syncing: VFS: Unable to mount root fs on unknown-block(1,0)
"
root file system이 없다면서 종료됨.
root file system만 생성해 주면 부팅 될 듯


여기서부터는 아래 사이트 참조.
https://medicineyeh.wordpress.com/2016/03/29/buildup-your-arm-image-for-qemu/

FHANDLE 적용
General setup  --->
[*] open by fhandle syscalls

devtmpfs 적용
Device Drivers  --->
  Generic Driver Options  --->
    [*] Maintain a devtmpfs filesystem to mount at /dev
    [*]   Automount devtmpfs at /dev, after the kernel mounted the rootfs

root file system을 만들기 위해 buildroot 사용
WSL에서 문제발생... 컴파일러 복사 에러


cpio 유틸로 파일시스템 생성

SYSV, V7, ROMFS 파일시스템 지원 추가
$ echo final_image.elf | cpio -o --format=newc > rootfs
qemu 실행할 때 initrd 옵션 추가
qemu-system-arm -machine integratorcp -kernel ./arch/arm/boot/zImage -m 256M -dtb ./arch/arm/boot/dts/integratorcp.dtb -serial stdio -initrd ../my_os/rootfs

아래 메시지를 출력하면서 커널패닉
"
[    1.222003] No filesystem could mount root, tried:  ext2 cramfs vfat sysv v7 romfs
"
생성된 rootfs가 현재 컴파일된 zImage에서 지원되지 않음
어떤게 지원이 되고 어떤게 지원이 안되는지?
모든 파일시스템 지원을 포함하면 되는지?

아니고, rootfs가 실행되는 명령을 추가했을 경우 hang 됨.
-append "root=/dev/ram rdinit=main"


컴파일러가 문제다....
arm-none-eabi-gcc 로 컴파일 했을 때는 제대로된 실행파일이 생성되지 않음.
윈도우 버전의 컴파일러라 그랬을 수 있나??? 커널은 이걸로 컴파일 했는데???

arm-linux-gnueabi-gcc 로 컴파일 후에 
https://landley.net/writing/rootfs-howto.html
참조하여 initramfs 만든 후 실행하니 깔끔하게 잘 됨.

화면은 왜 안나올까???

Device Drivers
 Graphics support
  [*]Bootup logo
     Console display driver support --->
      <*> Framebuffer Console support
설정 후 커널컴파일 후 QEMU 재실행 하니 로고가 나옴.. 

성공!!

qemu-system-arm -machine integratorcp -kernel ./arch/arm/boot/zImage -m 256M -dtb ./arch/arm/boot/dts/integratorcp.dtb -serial stdio -initrd ../my_os/initramfs_data.cpio.gz -append "console=tty0 earlyprintk init=/init"
명령으로 실행하니 console에서 리눅스 부팅메시지 확인가능

완전 성공!!!
