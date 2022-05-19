# Build Minimal Linux OS with Busybox On Top Of ARM64 QEMU Virtualization

  

## Prepare Build Environment

  

Install necessary packages

  

````bash

sudo apt install gcc-aarch64-linux-gnu build-essential lzop bison libncurses-dev git make gcc fakeroot ncurses-dev xz-utils libssl-dev bc flex libelf-dev

````

  



  

Create a directory named qemu and go inside

````bash

mkdir qemu

cd qemu

````

  
  

## Download and Build Linux Kernel

  

1. Download linux kernel 4.19.18 from kernel.org

  

````bash

wget https://cdn.kernel.org/pub/linux/kernel/v4.x/linux-4.19.18.tar.gz

````

  

2. Extract kernel source

  

````bash

tar -xvf linux-4.19.18.tar.gz

````

  

3. Go inside extracted folder

  

````bash

cd linux-4.19.18

````

  

4. Make default config for kernel compilation

  

````bash

make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- defconfig

````

  

5. Open **.config** file

  

````bash

vim .config

````

  

6. Enable NVME & PCIE
> Search for the CONFIGS and make similar shown below. If any entry is not present, add that at the last of the file

  

````bash

vim .config

````

  

- CONFIG_CONFIGFS_FS=y

- CONFIG_NVME_CORE=y

- CONFIG_BLK_DEV_NVME=y

- CONFIG_NVME_TARGET=y

- CONFIG_RTC_NVMEM=y

- CONFIG_NVMEM=y

- CONFIG_KGDB =y

  

7. Make **.config** compatible

  

````bash

yes "" | make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- oldconfig -j$(nproc)

````

  

8. Start compile process

  

````bash

make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- -j$(nproc)

````

> ^-^

  

> Compilation will stop after a few seconds. Do rest of the works.

  

> ^-^

  

9. Open **scripts/dtc/dtc-lexer.lex.c**

  

````bash

vim scripts/dtc/dtc-lexer.lex.c

````

  

10. Search for **yylloc** & delete this line, save file, exit

````bash

YYLTYPE yylloc;

````

  

11. Start the compile process again

  

````bash

make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- -j$(nproc)

````

  

> ^~^

  

>  **Congratulations!! Your compiled kernel is located at arch/arm64/boot directory**

  

> ^~^

  

12. Issue this command to check it

  

````bash

ls -lah arch/arm64/boot/Image

````

  

13. Back to the parent directory

  

````bash

cd ../

````

  
  

## Download and Build Busybox & Create rootfs

  

1. Download Busybox 1.35.0

  

````bash

wget https://busybox.net/downloads/busybox-1.35.0.tar.bz2

````

  

2. Extract busybox source

  

````bash

tar -xvf busybox-1.35.0.tar.bz2

````

  

3. Go inside extracted folder

  

````bash

cd busybox-1.35.0

````

  

4. Modify build config

  

> Enable **Build static binary (no shared libs)**

  

````bash

make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- menuconfig -j$(nproc)

````

  

5. Start compilation

  

````bash

make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- -j$(nproc)

````

  

6. Make directory for rootfs

  

````bash

mkdir rootfs

````

  

7. Copy files and go inside rootfs

  

````bash

make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- install CONFIG_PREFIX=rootfs -j$(nproc)

cd rootfs

````

  

8. Make required directories in rootfs

  

````bash

mkdir -pv {bin,sbin,etc/init.d,proc,sys,usr/{bin,sbin}}

````

  

9. Create **etc/init.d/rcS** file

````bash

vim etc/init.d/rcS

````

  

19. Add following lines & save

````bash

#!/bin/sh

#This is the first script called by init process

/bin/mount -a

echo /sbin/mdev>/proc/sys/kernel/hotplug

mdev -s

````

11. Make **etc/init.d/rcS** exicutable

````bash

chmod a+x etc/init.d/rcS

````

  

12. Create **etc/fstab** file

````bash

vim etc/fstab

````

  

13. Add following lines & save

````bash

#device mount-point type options dump fsck order

proc /proc proc defaults 0 0

tmpfs /tmp tmpfs defaults 0 0

sysfs /sys sysfs defaults 0 0

tmpfs /dev tmpfs defaults 0 0

````

  

14. Build **rootfs.img**

````bash

find . -print0 | cpio --null -ov --format=newc | gzip -9 > ../../rootfs.img

````

  

15. Back to the parent directory

  

````bash

cd ../../

````

  

> ^~^

  

>  **Congratulations!! Your rootfs.img is ready to be fired.**

  

> ^~^

  

## Make Virtual NVME Drive & Launch QEMU

  

1. Create **nvme.img** of 4GB

  

````bash

dd if=/dev/zero of=nvme.img bs=1M count=4096

````

  

2. Bring up the system

  

````bash

qemu-system-aarch64 -M virt -cpu cortex-a53 -smp 2 -m 4096M -kernel linux-4.19.18/arch/arm64/boot/Image -nographic -append "console=ttyAMA0 rdinit=linuxrc" -initrd rootfs.img

````

  

> Exit QEMU by pressing Ctrl+A then X

  

3. Bring up the system with NVME

  

````bash

qemu-system-aarch64 -M virt -cpu cortex-a53 -smp 2 -m 4096M -kernel linux-4.19.18/arch/arm64/boot/Image -nographic -append "console=ttyAMA0 rdinit=linuxrc" -initrd rootfs.img -drive file=nvme.img,format=raw,if=none,id=nvme -device nvme,serial=deadbeef,drive=nvme

````

  

> ^~^

  

>  **Congratulations!! You are good to GO!!!**
````bash

uname -a

````

