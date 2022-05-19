# Linux Kernel Module

## Install Dependencies


```bash
sudo apt install make gcc vim build-essential kmod linux-headers-$(uname -r) pahole
sudo cp /sys/kernel/btf/vmlinux /usr/lib/modules/$(uname -r)/build/
```

## Module Source Code
Create a file in working directory named helloworld.c
```bash
vim helloworld.c
```
Write following code into the file then save.
```bash
#include <linux/init.h>
#include <linux/module.h>

static int __init init_hello(void) {
    printk(KERN_ALERT "Hello World!!\n");
    return 0;
}

static void __exit exit_hello(void) {
    printk(KERN_ALERT "Goodbye World!!\n");
    return;
}

module_init(init_hello);
module_exit(exit_hello);

MODULE_LICENSE("GPL");
```

Create another file in working directory named Makefile
```bash
vim Makefile
```
Write following code into the file then save.
```bash
obj-m:= helloworld.o

PWD := $(CURDIR)

all:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules

clean:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean
```

Now run "make" from your terminal. It will produce a "helloworld.ko" file. Load it into kernel by running
```bash
sudo insmod ./helloworld.ko
```

Examine kernel log.

```bash
sudo dmesg
```

Unload the module by running
```bash
sudo rmmod helloworld
```

Examine kernel log.

```bash
sudo dmesg
```

