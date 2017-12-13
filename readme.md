������ Ŀ�� ����
make ARCH=arm menuconfig

printk ����
 - Kernel hacking --> printk and dmesg options --> Show timing information on printks
CONFIG_DEBUG_LL
 - Kernel hacking -> Kernel low-level debugging functions
CONFIG_DEBUG_LL_UART_PL01X (CONFIG_DEBUG_LL)
                  -> Kernel low-level debugging port -> Kernel low-level debugging via ARM Ltd PL01x Primecell
CONFIG_EARLY_PRINTK (CONFIG_DEBUG_LL)
 - Kernel hacking -> Early printk


zImage ������
make ARCH=arm CROSS_COMPILE=arm-none-eabi- -j2 zImage


QEMU ����
qemu-system-arm -machine integratorcp -kernel ./arch/arm/boot/zImage -m 256M

QEMU���� Ctrl+Alt+3 ������ serial0 console�� ��ȯ����
serial0 console���� ������ ���� ��µǸ鼭 ������
"
Uncompressing Linux... done, booting the kernel.
no ATAGS support: can't continue
"
ATAGS �ɼ� ������ �ʿ��� ��

CONFIG_ATAGS
Boot options -> Support for the tradditional ATAGS boot data passing


�ٽ� zImage ������ �� QMEMU ����
�����߻�
"
Error: unrecognized/unsupported machine ID (r1 = 0x00000113).

Available machine support:

ID (hex)        NAME
ffffffff        Generic DT based system
ffffffff        ARM Integrator/AP (Device Tree)
ffffffff        ARM Integrator/CP (Device Tree)

Please check your kernel config and/or bootloader.
"

Ŀ�� ������ (DTB���� ����)
make ARCH=arm CROSS_COMPILE=arm-none-eabi- -j2


QEMU ����� DTB�ɼ� ����
qemu-system-arm -machine integratorcp -kernel ./arch/arm/boot/zImage -m 256M -dtb ./arch/arm/boot/dts/integratorcp.dtb -serial stdio
"
---[ end Kernel panic - not syncing: VFS: Unable to mount root fs on unknown-block(1,0)
"
root file system�� ���ٸ鼭 �����.
root file system�� ������ �ָ� ���� �� ��


���⼭���ʹ� �Ʒ� ����Ʈ ����.
https://medicineyeh.wordpress.com/2016/03/29/buildup-your-arm-image-for-qemu/

FHANDLE ����
General setup  --->
[*] open by fhandle syscalls

devtmpfs ����
Device Drivers  --->
  Generic Driver Options  --->
    [*] Maintain a devtmpfs filesystem to mount at /dev
    [*]   Automount devtmpfs at /dev, after the kernel mounted the rootfs

root file system�� ����� ���� buildroot ���
WSL���� �����߻�... �����Ϸ� ���� ����


cpio ��ƿ�� ���Ͻý��� ����

SYSV, V7, ROMFS ���Ͻý��� ���� �߰�
$ echo final_image.elf | cpio -o --format=newc > rootfs
qemu ������ �� initrd �ɼ� �߰�
qemu-system-arm -machine integratorcp -kernel ./arch/arm/boot/zImage -m 256M -dtb ./arch/arm/boot/dts/integratorcp.dtb -serial stdio -initrd ../my_os/rootfs

�Ʒ� �޽����� ����ϸ鼭 Ŀ���д�
"
[    1.222003] No filesystem could mount root, tried:  ext2 cramfs vfat sysv v7 romfs
"
������ rootfs�� ���� �����ϵ� zImage���� �������� ����
��� ������ �ǰ� ��� ������ �ȵǴ���?
��� ���Ͻý��� ������ �����ϸ� �Ǵ���?

�ƴϰ�, rootfs�� ����Ǵ� ����� �߰����� ��� hang ��.
-append "root=/dev/ram rdinit=main"


�����Ϸ��� ������....
arm-none-eabi-gcc �� ������ ���� ���� ����ε� ���������� �������� ����.
������ ������ �����Ϸ��� �׷��� �� �ֳ�??? Ŀ���� �̰ɷ� ������ �ߴµ�???

arm-linux-gnueabi-gcc �� ������ �Ŀ� 
https://landley.net/writing/rootfs-howto.html
�����Ͽ� initramfs ���� �� �����ϴ� ����ϰ� �� ��.

ȭ���� �� �ȳ��ñ�???

Device Drivers
 Graphics support
  [*]Bootup logo
     Console display driver support --->
      <*> Framebuffer Console support
���� �� Ŀ�������� �� QEMU ����� �ϴ� �ΰ� ����.. 

����!!

qemu-system-arm -machine integratorcp -kernel ./arch/arm/boot/zImage -m 256M -dtb ./arch/arm/boot/dts/integratorcp.dtb -serial stdio -initrd ../my_os/initramfs_data.cpio.gz -append "console=tty0 earlyprintk init=/init"
������� �����ϴ� console���� ������ ���ø޽��� Ȯ�ΰ���

���� ����!!!
