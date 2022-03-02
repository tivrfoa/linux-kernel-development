
## Clonning the code

`git clone git://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git`

## Configuring

`make localmodconfig`

https://www.kernel.org/doc/Documentation/admin-guide/README.rst
```txt
     "make localmodconfig" Create a config based on current config and
                           loaded modules (lsmod). Disables any module
                           option that is not needed for the loaded modules.

                           To create a localmodconfig for another machine,
                           store the lsmod of that machine into a file
                           and pass it in as a LSMOD parameter.

                           Also, you can preserve modules in certain folders
                           or kconfig files by specifying their paths in
                           parameter LMC_KEEP.

                   target$ lsmod > /tmp/mylsmod
                   target$ scp /tmp/mylsmod host:/tmp

                   host$ make LSMOD=/tmp/mylsmod \
                           LMC_KEEP="drivers/usb:drivers/gpu:fs" \
                           localmodconfig

                           The above also works when cross compiling.
```

More info here: https://www.kernel.org/doc/Documentation/kbuild/kconfig.rst

## Compiling

`make -j$(nproc)`

## Creating init RAM disk

`mkinitramfs -o ramdisk.img`

>The basic initramfs is the root filesystem image used for booting the kernel provided as a compressed cpio archive.

https://wiki.debian.org/initramfs

## Running in Qemu

*Run the commands from the Linux folder*

`qemu-system-x86_64 -kernel arch/x86_64/boot/bzImage -initrd ramdisk.img`

You'll get the error: not syncing: VFS: Unable to mount root fs on unkown-block

The error above happened because it didn't pass the parameter: -m 512

`qemu-system-x86_64 -kernel arch/x86_64/boot/bzImage -initrd ramdisk.img -m 512`

## Exiting Qemu

`Ctrl+A x`

## Show output on host

`qemu-system-x86_64 -kernel arch/x86_64/boot/bzImage -initrd ramdisk.img -nographic -m 512 -append console=ttyS0`


## Qemu GDB

>The -s option will make QEMU listen for an incoming connection from gdb on TCP port 1234, and -S will make QEMU not start the guest until you tell it to from gdb.
> source: https://www.qemu.org/docs/master/system/gdb.html
>

`qemu-system-x86_64 -s -S -kernel arch/x86_64/boot/bzImage -initrd ramdisk.img -nographic -m 512 -append console=ttyS0`

bzImage doesn't contain debugging symbols, so we must use vmlinux

```
$ gdb arch/x86_64/boot/bzImage
GNU gdb (Ubuntu 10.2-0ubuntu1~20.04~1) 10.2
Copyright (C) 2021 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Type "show copying" and "show warranty" for details.
This GDB was configured as "x86_64-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<https://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.

For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from arch/x86_64/boot/bzImage...
(No debugging symbols found in arch/x86_64/boot/bzImage)
(gdb) target remote :1234
Remote debugging using :1234
0x000000000000fff0 in ?? ()
(gdb) Quit
(gdb) quit
A debugging session is active.

	Inferior 1 [process 1] will be detached.

Quit anyway? (y or n) y
Detaching from program: /home/lesco/dev/github/linux/arch/x86_64/boot/bzImage, process 1
Ending remote debugging.
[Inferior 1 (process 1) detached]
```

#### Using vmlinux

`qemu-system-x86_64 -s -S -kernel vmlinux -initrd ramdisk.img -nographic -m 512 -append console=ttyS0`

```
$ gdb vmlinux
GNU gdb (Ubuntu 10.2-0ubuntu1~20.04~1) 10.2
Copyright (C) 2021 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Type "show copying" and "show warranty" for details.
This GDB was configured as "x86_64-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<https://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.

For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from vmlinux...
warning: File "/home/lesco/dev/github/linux/scripts/gdb/vmlinux-gdb.py" auto-loading has been declined by your `auto-load safe-path' set to "$debugdir:$datadir/auto-load".
To enable execution of this file add
	add-auto-load-safe-path /home/lesco/dev/github/linux/scripts/gdb/vmlinux-gdb.py
line to your configuration file "/home/lesco/.gdbinit".
To completely disable this security protection add
	set auto-load safe-path /
line to your configuration file "/home/lesco/.gdbinit".
For more information about this security protection see the
"Auto-loading safe path" section in the GDB manual.  E.g., run from the shell:
	info "(gdb)Auto-loading safe path"
```

As we saw in the error above, we must first enable vmlinux-gdb.py

Let's try again:

```sh
linux (master)$ gdb vmlinux
GNU gdb (Ubuntu 10.2-0ubuntu1~20.04~1) 10.2
Copyright (C) 2021 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Type "show copying" and "show warranty" for details.
This GDB was configured as "x86_64-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<https://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.

For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from vmlinux...
(gdb) target remote :1234
Remote debugging using :1234
0x000000000000fff0 in exception_stacks ()
(gdb) list 500
495
496		return &cpu_base->clock_base[idx];
497	}
498
499	#define for_each_active_base(base, cpu_base, active)	\
500		while ((base = __next_base((cpu_base), &(active))))
501
502	static ktime_t __hrtimer_next_event_base(struct hrtimer_cpu_base *cpu_base,
503						 const struct hrtimer *exclude,
504						 unsigned int active,
(gdb) info threads
  Id   Target Id                    Frame
* 1    Thread 1.1 (CPU#0 [running]) 0x000000000000fff0 in exception_stacks ()
(gdb) b start_kernel 
Breakpoint 1 at 0xffffffff82fc0fac: file init/main.c, line 928.
(gdb) c
Continuing.

Breakpoint 1, start_kernel () at init/main.c:928
928	{
(gdb) list 200
195	extern const struct obs_kernel_param __setup_start[], __setup_end[];
196	
197	static bool __init obsolete_checksetup(char *line)
198	{
199		const struct obs_kernel_param *p;
200		bool had_early_param = false;
201	
202		p = __setup_start;
203		do {
204			int n = strlen(p->str);

```

## References

https://pnx9.github.io/thehive/Debugging-Linux-Kernel.html

https://www.qemu.org/docs/master/system/gdb.html

https://wiki.gentoo.org/wiki/QEMU/Options

https://superuser.com/questions/1087859/how-to-quit-the-qemu-monitor-when-not-using-a-gui



