---
title: "创建一个简单的内核模块"
date: 2023-10-07T19:33:16+08:00
categories: ["The Operating System"]
tags: ["os", "linux", "kernel"]
draft: false
---

# 创建一个简单的内核模块
内核模块文件：
```c
#include <linux/module.h>
#include <linux/init.h>

static int __init my_test_init(void)
{
	printk(KERN_EMERG "my first kernel module init\n");
	return 0;
}

static void __exit my_test_exit(void)
{
	printk("goodbye\n");
}

module_init(my_test_init);
module_exit(my_test_exit);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("rlk");
MODULE_DESCRIPTION("my test kernel module");
MODULE_ALIAS("mytest");
```
1. `<linux/init.h>` 对应的是 `include/linux/init.h`，包含了 `module_init()` 和 `module_exit()` 的函数声明。`module_init` 表示这是模块入口，`module_exit` 告诉内核该模块的退出函数。

2. `<linux/module.h>` 对应的是 `include/linux/module.h` 包含了 `MODULE_AUTHOR()` 等一些宏的声明。

接着来看如何编译内核模块：
```Makefile
#BASEINCLUDE ?= /home/rlk/rlk/runninglinuxkernel_5.0
BASEINCLUDE ?= /lib/modules/`uname -r`/build

CONFIG_MODULE_SIG=n

mytest-objs := my_test.o 

obj-m	:=   mytest.o
all : 
	$(MAKE) -C $(BASEINCLUDE) M=$(PWD) modules;

clean:
	$(MAKE) -C $(BASEINCLUDE) M=$(PWD) clean;
	rm -f *.ko;

```

1. `BASEINCLUDE` 指向正在运行 Linux 的内核编译目录。
2. `<模块名>-objs := <目标文件>.o` 表示该内核模块需要哪些目标文件
3. `obj-m := <模块名>.o` 表示要生成的模块

插入模块:
```shell
sudo insmod mytest.ko
```
查看模块信息：
```shell
modinfo mytest.ko
```
查看内核的输出信息：
```shell
dmesg
```
查看有哪些模块被加载：
```shell
lsmod
```
卸载模块：
```shell
rmmod mytest
```
# 模块参数
```c
#define module_param(name, type, perm) \
    module_param_named(name, name, type, perm)

#define MODULE_PARM_DESC(_parm, desc) \
    __MODULE_INFO(parm, _parm, #_parm ":" desc)
```
1. `module_param()` 宏有三个参数：name 表示参数名，type 表示参数类型，perm 表示参数的读写等权限。
2. `MODULE_PARM_DESC` 宏为参数做了简单类型

例子：
```c
#include <linux/module.h>
#include <linux/init.h>

static int debug = 1;
module_param(debug, int, 0644);
MODULE_PARM_DESC(debug, "enable debugging information");

#define dprintk(args...) \
	if (debug) { \
		printk(KERN_DEBUG args); \
	}	


static int mytest = 100;
module_param(mytest, int, 0644);
MODULE_PARM_DESC(mytest, "test for module parameter");

static int __init my_test_init(void)
{
	dprintk("my first kernel module init\n");
	dprintk("module parameter=%d\n", mytest);
	return 0;
}

static void __exit my_test_exit(void)
{
	printk("goodbye\n");
}

module_init(my_test_init);
module_exit(my_test_exit);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("kylin");
MODULE_DESCRIPTION("my test kernel module");
MODULE_ALIAS("mytest");
```

# 符号共享
使用下面的宏来导出
```c
EXPORT_SYMBOL()
EXPORT_SYMBOL_GPL()
```
`sudo cat /proc/kallsyms` 来查看内核导出的符号表