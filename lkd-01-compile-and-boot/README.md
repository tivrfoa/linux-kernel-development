

# Linux Kernel Programming 01: Compile and Boot


[https://www.youtube.com/watch?v=WiZ05pnHZqM](https://www.youtube.com/watch?v=WiZ05pnHZqM)


## Download the kernel

```sh
git clone --depth=10 https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/
```

## Open the kernel source in kdevelop

```sh
sudo apt install kdvelop
```

Project -> Open/Import project ...

then select the folder that you cloned.

### Include /include directory to path

Configure Project -> Configure Language Support ->
Includes/Imports


### Linux main function

linux/init/main.c

```c
asmlinkage __visible void __init __no_sanitize_address start_kernel(void)
{
	char *command_line;
	char *after_dashes;

	set_task_stack_end_magic(&init_task);
	smp_setup_processor_id();
	debug_objects_early_init();
	init_vmlinux_build_id();

	cgroup_init_early();
	//...
}
```

## Configuration

>The best option is to take the `.config` file from your
distribution.


```sh
cp /boot/config-$(uname -r) .config
```

### Editing config

```sh
make menuconfig
```

or

```sh
make xconfig
```

### Updating config file to the new kernel version

```sh
make oldconfig
```

## Compiling

```sh
make -j$(nproc)
```

## Installing modules

```sh
sudo make modules_install
```

