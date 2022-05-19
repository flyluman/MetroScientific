# Write System Call In Linux

## Prepare Build Environment

Install necessary packages.

```sh
sudo apt install git make gcc fakeroot build-essential ncurses-dev xz-utils libssl-dev bc flex libelf-dev bison libncurses-dev
```
## Download & Extract Kernel Source

```sh
wget https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.17.7.tar.xz
tar xvf linux-5.17.7.tar.xz
cd linux-5.17.7
```

## Write System Call Source Code

Create a directory to hold system call source code.
```sh
mkdir helloworld
```

Create system call source code.
```sh
vim helloworld/helloworld.c
```
Put this in "helloworld.c" & save.
```sh
#include <linux/kernel.h>
#include <linux/syscalls.h>

SYSCALL_DEFINE0(helloworld) {
	printk("Hello World!!\n");
	return 0;
}
```

Create helloworld Makefile.

```sh
vim helloworld/Makefile
```
Put this in helloworld Makefile & save.
```sh
obj-y:= helloworld.o

```

## Connect Our System Call To The Kernel

Open Makefile.
```sh
vim Makefile
```

Search for crypto/

Add helloworld/ in that line after crypto/ & save.

It should look like this.
````sh
core-y			+= kernel/ certs/ mm/ fs/ ipc/ security/ crypto/ helloworld/
````

Open "include/linux/syscalls.h" file.
```sh
vim include/linux/syscalls.h
```
Add this line at the bottom of the file & save.
```sh
asmlinkage long sys_helloworld(void);
```

Open "arch/x86/entry/syscalls/syscall_64.tbl" file.
```sh
vim arch/x86/entry/syscalls/syscall_64.tbl
```
Add "335	common	helloworld		sys_helloworld" after "334	common	rseq			sys_rseq" of the file & save.

It should look like this.
```sh
334	common	rseq			sys_rseq
335	common	helloworld		sys_helloworld
```

## Compile The Kernel

Clean Build Directory.
````bash
make clean
make mrproper
````

Download & Rename .config
````bash
wget https://raw.github.com/xanmod/linux/5.17/CONFIGS/xanmod/gcc/config_x86-64
mv config_x86-64 .config
````

Make ".config" compatible.
````bash
scripts/config --disable SYSTEM_REVOCATION_KEYS
scripts/config --disable SYSTEM_TRUSTED_KEYS
scripts/config --disable CONFIG_X86_X32
yes "" | make oldconfig
````

Start the build.
````bash
make -j$(nproc)
````

## Install The Modules & Kernel

````bash
sudo make modules_install -j$(nproc)
sudo make install -j$(nproc)
````

Reboot to the new kernel.

````bash
reboot
````

## Test Our System Call

Create a file named "test.c".
````bash
vim test.c
````
Put this in test.c and save.
````bash
#include <linux/kernel.h>
#include <sys/syscall.h>
#include <stdio.h>
#include <unistd.h>
#include <string.h>
#include <errno.h>

#define __NR_helloworld 335

long helloworld_syscall(void)
{
    return syscall(__NR_helloworld);
}

int main(int argc, char *argv[])
{
    long activity;
    activity = helloworld_syscall();

    if(activity < 0)
    {
        perror("Your system call appears to have failed.");
    }

    else
    {
        printf("Congratulations! Your system call is functional. Run the command dmesg in the terminal and find out more!\n");
    }

    return 0;
}
````

Compile & Run "test".
````bash
gcc test.c -o test
./test
````

Examine Output :)

Examine dmesg :)
````bash
sudo dmesg
````
