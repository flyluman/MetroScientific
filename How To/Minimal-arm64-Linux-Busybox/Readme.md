# Build Minimal Linux OS with Busybox On Top Of ARM64 QEMU Virtualization

  

## Prepare Build Environment

  

Install necessary packages

  

````bash

sudo apt install qemu-system-arm qemu-system-aarch64 qemu-utils gcc-aarch64-linux-gnu build-essential lzop bison libncurses-dev git make gcc fakeroot ncurses-dev xz-utils libssl-dev bc flex libelf-dev

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

5. Make **.config** compatible for virtual environment

  

````bash

make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- kvmconfig

````

6. Enable All NVME configs

  

````bash

make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- menuconfig

````

  

7. Make **.config** compatible

  

````bash

yes "" | make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- oldconfig

````

  

8. Start compile process

  

````bash

make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- Image -j$(nproc)

````

  

>  **Congratulations!! Your compiled kernel is located at arch/arm64/boot directory**

  

> ^~^

  

9. Issue this command to check it

  

````bash

ls -lah arch/arm64/boot/Image

````

  

10. Back to the parent directory

  

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

  

> Enable **Settings > Build Options > Build static binary (no shared libs)**

  

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

mkdir -pv {bin,sbin,etc,proc,sys,usr/{bin,sbin}}
````

  

9. Create **init** file

````bash

vim init

````

  

19. Add following lines & save

````bash

#!/bin/sh
 
mount -t proc none /proc
mount -t sysfs none /sys

#Create device nodes
mknod /dev/null c 1 3
mknod /dev/tty c 5 0
mdev -s
 
echo -e "\nBoot took $(cut -d' ' -f1 /proc/uptime) seconds\n"
 
exec /bin/sh

````

11. Make **init** exicutable

````bash

chmod a+x init

````

  

12. Build **rootfs.img**

````bash

find . -print0 | cpio --null -ov --format=newc | gzip -9 > ../../rootfs.img

````

  

13. Back to the parent directory

  

````bash

cd ../../

````

  

> ^~^

  

>  **Congratulations!! Your rootfs.img is ready to be fired.**

  

> ^~^

  

## Make Virtual NVME Drive & Launch QEMU

  

1. Create **nvme.img** of 512MB

  

````bash

dd if=/dev/zero of=nvme.img bs=1M count=512

````

  

2. Bring up the system

  

````bash

qemu-system-aarch64 -M virt -cpu cortex-a53 -smp 2 -m 512M -kernel linux-4.19.18/arch/arm64/boot/Image -nographic -append "console=ttyAMA0 init=/init" -initrd rootfs.img

````

  

> Exit QEMU by pressing Ctrl+A then X

  

3. Bring up the system with NVME

  

````bash

qemu-system-aarch64 -M virt -cpu cortex-a53 -smp 2 -m 512M -kernel linux-4.19.18/arch/arm64/boot/Image -nographic -append "console=ttyAMA0 init=/init" -initrd rootfs.img -drive file=nvme.img,format=raw,if=none,id=nvme -device nvme,serial=deadbeef,drive=nvme

````

  

> ^~^

  

>  **Congratulations!! You are good to GO!!!**
````bash

uname -a

````

