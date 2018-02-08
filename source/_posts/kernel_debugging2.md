title: Linux 内核学习笔记（二）调试、扩展
date: 2018-02-01
tags: kernel linux
categories: kernel
---

  上一节我们研究了如何编译内核和构建rootfs，以及编译模块 ，这使得我们构建了一个足够单纯、自由的实验环境，本节讲主要围绕模块的具体构建方式和内核的调试方式。希望通过这节之后我们可以和内核有一个真正的互动。

## 内核日志级别

  首先为了可以看到更详细的系统日志，我们需要可以有一个手动设置当前内核日志级别的方式，当然你可以在编译内核的时候设定他的默认日志级别，或者在你的init脚本里面设定。为了设定日志级别我们需要首先挂载proc文件系统，然后把日志级别写入到/proc/sys/kernel/printk中，cat /proc/sys/kernel/printk可以查看当前的日志级别，具体的含义还请使用man 2 syslog查看，这里关注第一个参数打印到终端的最低级别（数字越大级别越低)

```

mount -t proc proc /proc
echo 7 > /proc/sys/kernel/printk

```

另外为了后续的调试方便可以顺便把sysfs也挂载
  
```

mount -t sysfs sysfs /sys

```

之后开启日志相关守护进程即可在/var/log/messages中看到系统日志

```

/sbin/klogd
/sbin/ksyslogd

```
  

## GDBServer调试

在使用 gdb加载vmlinux的时候可能会有以下警告，一些发布版本处于安全考虑可能禁用了gdb导入python脚本的功能，具体可以查看Documentation/gdb-kernel-debugging.txt 当中的说明，我们这里按照他的提示把 set auto-load safe-path / 放到home下的.gdbinit先屏蔽掉这个保护功能

```
Reading symbols from vmlinux...done.
warning: File "/home/chenpeng/Documents/linux/scripts/gdb/vmlinux-gdb.py" auto-loading has been declined by your `auto-load safe-path' set to "$debugdir:$datadir/auto-load".
To enable execution of this file add
	add-auto-load-safe-path /home/chenpeng/Documents/linux/scripts/gdb/vmlinux-gdb.py
line to your configuration file "/home/chenpeng/.gdbinit".
To completely disable this security protection add
	set auto-load safe-path /
line to your configuration file "/home/chenpeng/.gdbinit".
For more information about this security protection see the
"Auto-loading safe path" section in the GDB manual.  E.g., run from the shell:
	info "(gdb)Auto-loading safe path"
```
这样在gdb启动后会加载vmlinux-gdb.py的内容，它通过一些Python的脚本扩展了gdb的功能，主要是增加了lx-系列的命令。

## GDB调试内核模块
    经过前面加载了 vmlinux-gdb.py 之后，我们可以使用 lx-symbols /path/to/module/build 来在gdb中给予调试的vmlinux加载模块的符号，从而实现可以调试模块和在模块代码中打断点。
这里注意首先启动gdb的权限需要有读写 /path/to/module/build 的权限，其次只有在客户环境中使用 insmod加载模块后，才会触发gdb导入其中的符号。
再有，我这里 /path/to/module/build 路径必须是绝对路径
    这里还有一点由于在内核的配置选项中找不到和优化级别相关的配置，我擅自把Makefile中的所有-On参数全部改成了-O1，因为担心过高的优化级别会影响后续的调试，但是-O0是无法通过编译的，原因暂时没有深究,不知道后面会不会有问题。

```

Reading symbols from vmlinux...done.
(gdb) lx-symbols /home/chenpeng/qemu_linux/rootfs/kernel_hacking
loading vmlinux
(gdb) target remote localhost:1234
Remote debugging using localhost:1234
native_safe_halt () at ./arch/x86/include/asm/irqflags.h:50
50	}
(gdb) c
Continuing.   #######这里在客户环境 insmod hello.ko
scanning for modules in /home/chenpeng/qemu_linux/rootfs/kernel_hacking
scanning for modules in /home/chenpeng/Documents/linux
loading @0xffffffffc0005000: /home/chenpeng/qemu_linux/rootfs/kernel_hacking/hello.ko

C-c C-c
Program received signal SIGINT, Interrupt.
native_safe_halt () at ./arch/x86/include/asm/irqflags.h:50
50	}
(gdb) b hello_exit
Breakpoint 1 at 0xffffffffc000503a: file /home/chenpeng/qemu_linux/rootfs/kernel_hacking/hello.c, line 14.

```


## 模块的具体内容


