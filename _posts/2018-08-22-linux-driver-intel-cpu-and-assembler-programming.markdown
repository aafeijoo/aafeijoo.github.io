---
layout: post
title: "Linux driver, Intel CPU and assembler programming"
date: 2018-08-22 19:10:42
categories: source code
tags: c assembler linux intel driver
image: /assets/images/articles/2018-08-22-linux-driver-intel-cpu-and-assembler-programming/linux-driver-intel-cpu-and-assembler-programming.png
---

Some time ago I was playing with Linux drivers after attending to a great course from [ElectraTraining](http://www.electratraining.org/english/). I wanted to develop a driver that replaces some handler in the system call table (read-only) to intercept that system call.

To be able to modify the system call table when running Linux on an Intel CPU, we have to modify the register CR0.

> CR0.WP
> Write Protect (bit 16 of CR0) — When set, inhibits supervisor-level procedures from writing into readonly
> pages; when clear, allows supervisor-level procedures to write into read-only pages (regardless of the
> U/S bit setting; see Section 4.1.3 and Section 4.6). This flag facilitates implementation of the copy-onwrite
> method of creating a new process (forking) used by operating systems such as UNIX.

The way of modifying the register CR0 is using the `mov` assembler instruction.

> The control registers can be read and loaded (or modified) using the move-to-or-from-control-registers forms
> of the MOV instruction. In protected mode, the MOV instructions allow the control registers to be read or loaded
> (at privilege level 0 only). This restriction means that application programs or operating-system procedures
> (running at privilege levels 1, 2, or 3) are prevented from reading or loading the control registers.

In Linux, sometimes we have to work in kernel space to be able to perform tasks that are not allowed in user space. That's one of the reasons why drivers exist.

![Privilege Rings](https://upload.wikimedia.org/wikipedia/commons/2/2f/Priv_rings.svg)

An example of all of this working together:

* I have modified the handler of the `__NR_open` key, which is called when an user application opens a file.
* The keys of the system call table are defined in the header `asm/unistd.h`.
* The function handlers are defined in the header `linux/syscalls.h`.
* After installing the module (`sudo insmod testmod.ko`), check the output of the dmesg command (`dmesg -wH`). Did you know that there were so many calls to the `open` function...?

{% highlight c %}
#include <linux/module.h>
#include <linux/kallsyms.h>
#include <linux/uaccess.h>
#include <asm/unistd.h>

static void **sys_call_table = NULL;

static asmlinkage long (*original_open)(const char __user *filename, int flags, umode_t mode);

static asmlinkage long testmod_open(const char __user *filename, int flags, umode_t mode)
{
    pr_info("[open] %s : %s\n", current->comm, filename);
    return original_open(filename, flags, mode);
}

static void disable_wp(void) 
{
    unsigned long value;
    asm volatile("mov %%cr0, %0" : "=r" (value));
    if (value & 0x00010000)
    {
        asm volatile("mov %0, %%cr0" : : "r" (value & ~0x00010000));
    }
}

static void enable_wp(void) 
{
    unsigned long value;
    asm volatile("mov %%cr0, %0" : "=r" (value));
    if (!(value & 0x00010000))
    {
        asm volatile("mov %0, %%cr0" : : "r" (value | 0x00010000));
    }
}

int init_module(void)
{
    sys_call_table = (void **)kallsyms_lookup_name("sys_call_table");
    if (!sys_call_table)
    {
        pr_err("testmod: can't find sys_call_table symbol\n");
        return -ENXIO;
    }

    disable_wp(); 
    {
        original_open = sys_call_table[__NR_open];
        sys_call_table[__NR_open] = testmod_open;
    }
    enable_wp();

    pr_info("testmod: loaded\n");
    return 0;
}

void cleanup_module(void)
{
    disable_wp(); 
    {
        sys_call_table[__NR_open] = original_open;
    }
    enable_wp();

    pr_info("testmod: unloaded\n");
}

MODULE_AUTHOR("Antonio Alvarez Feijoo");
MODULE_LICENSE("GPL");
MODULE_DESCRIPTION("Test kernel module that modifies the system call table");
{% endhighlight %}

References:

* [Intel® 64 and IA-32 Architectures Software Developer Manuals](https://software.intel.com/en-us/articles/intel-sdm)
* [Wikipedia](https://en.wikipedia.org/wiki/Protection_ring)
