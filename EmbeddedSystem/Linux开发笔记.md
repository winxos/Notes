# Linux 开发笔记

winxos 20200612

### 基础知识

### bootloader

### 交叉编译环境搭建

### 内核裁剪

### 网络编程

### 驱动开发

#### 字符设备驱动

```c
#include <linux/init.h>
#include <linux/module.h>
#include <linux/fs.h>
#include <linux/cdev.h>
#include <linux/slab.h>
#include <linux/uaccess.h>
#include <wiringPi.h>

MODULE_LICENSE("Dual BSD/GPL");
MODULE_AUTHOR("WVV");
static dev_t devno;
static struct class *cls;
static struct device *test;
#define IO_M 'w'
#define IO_INIT _IO(IO_M,0)
#define IO_R _IOR(IO_M,1,int)
#define IO_W _IOW(IO_M,2,int)
static int _open(struct inode *inode, struct file *f) {
    printk("hello open\n");
    return 0;
}

static int _release(struct inode *inode, struct file *f) {
    printk("hello release\n");
    return 0;
}
static long _ioctl(struct file *f,unsigned int cmd,unsigned  long arg)
{
    printk("hello ioctl %d\n",cmd);
    if(_IOC_TYPE(cmd)!=IO_M)
    {
        return -ENOTTY;
    }

    switch(cmd){
        case IO_INIT:
            printk("hello ioctl 0\n");
            break;
        case IO_R:
            printk("hello ioctl 1 %ld\n",arg);
            break;
        case IO_W:
            printk("hello ioctl 2 %ld\n",arg);
            break;
        default:
            break;
    }
    return 0;
}
ssize_t _read(struct file *f, char __user *buf,size_t count, loff_t *f_pos) {
    printk("hello read\n");
    char *b = kmalloc(count + 1, GFP_KERNEL);
    if (b == NULL) {

        return 0;
    }
    memset(b,'w', count);
    int ret = 0;
    if (copy_to_user(buf, b, count)) {
        printk("hello read error\n");
        ret = -EFAULT;
    } else {
        ret = count;
    }
    kfree(b);
    return ret;
}

static ssize_t _write(struct file *f, const char __user *buf, size_t count, loff_t *f_pos) {
    printk("hello write %s\n",buf);
    return 0;
}

static struct file_operations _ops = {
        .open=_open,
        .release=_release,
        .owner= THIS_MODULE,
        .read=_read,
        .write=_write,
        .unlocked_ioctl=_ioctl,
};
static int _hinit(void) {
    int ret = 0;
	wiringPiSetup ();
	pinMode (1, OUTPUT);
	printk(KERN_ALERT"hello init\n");
    devno = MKDEV(200, 0);
    ret = register_chrdev(200, "hello", &_ops);
    cls = class_create(THIS_MODULE, "hclass");
    if (IS_ERR(cls)) {
        unregister_chrdev(200, "hello");
        printk(KERN_ALERT
        "hello register fail\n");
        return ret;
    }
    test = device_create(cls, NULL, devno, NULL, "hello");
    if (IS_ERR(cls)) {
        class_destroy(cls);
        unregister_chrdev(200, "hello");
        printk(KERN_ALERT"hello add fail\n");
        return ret;
    }
	digitalWrite (1,HIGH);
    return 0;
}

static void _hexit(void) {
    printk(KERN_ALERT"hello exit\n");
    device_destroy(cls, devno);
    class_destroy(cls);
    unregister_chrdev(200, "hello");
	digitalWrite (1,LOW);
}

module_init(_hinit);
module_exit(_hexit);
```

内核文件 hello.c

```makefile
ifeq ($(KERNELRELEASE),)

KDIR := /lib/modules/$(shell uname -r)/build
PWD := $(shell pwd)
modules:
	$(MAKE) -C $(KDIR) M=$(PWD) -lwiringPi modules

modules_install:
	$(MAKE) -C $(KDIR) M=$(PWD) modules_install

clean:
	rm -rf *.o .depend *.mod.o *.mod.c Module.* modules.*

.PHONY:modules modules_install clean

else

obj-m :=hello.o

endif

```

Makefile

编译完成后会生成 hello.ko文件，这就是编译好的内核文件。

可以通过以下命令操作内核文件

```bash
sudo insmod hello.ko
lsmod
sudo rmmod hello
dmesg
```

insmod 插入内核

lsmod 列出加载好的内核

rmmod 移除内核

dmesg 查看内核调试信息

说明：

1. 上述代码实现了树莓派上一个字符驱动开发，模块挂载后用户可以通过文件操作来控制一个led灯。
2. Linux 内核2.6以后采用了设备树的挂载机制，设备的注册方式是不同的。
3. 上述代码是采用了手工指定设备号为200，也可以让系统自动分配设备号。
4. module_init和module_exit是为了让模块自动挂载和退出时自动退出。
5. 注意在` _ioctl`接口实现时，内核提供了一系列的宏，例如` _IOR`等, 用于规范CMD格式，自行设计cmd时，应该仔细理解其设计意图。
6. 注意内核2.6.36以后删除了ioctl接口，只保留了unlocked_ioctl接口。
7. 在read，write操作时，要注意内核空间和用户空间的区别，他们之间不能直接访问，需要使用`copy_to_user`等函数完成用户空间到内核空间的数据转换。

