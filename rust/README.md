# Rust for Linux

https://github.com/Rust-for-Linux/linux

https://github.com/Rust-for-Linux/linux/blob/rust/Documentation/rust/quick-start.rst


## Checking requirements

Running `make LLVM=1 rustavailable` I got:

```
***
*** Rust bindings generator 'bindgen' could not be found.
***
make: *** [Makefile:1779: rustavailable] Error 1
```

Installing `rust-bindgen`: `cargo install bindgen`

https://github.com/rust-lang/rust-bindgen/blob/master/book/src/command-line-usage.md

On Ubuntu you can install bindgen with the following command:
```sh
sudo apt install llvm-dev libclang-dev clang
```

https://rust-lang.github.io/rust-bindgen/requirements.html

Then I got this error:

```sh
$ make LLVM=1 rustavailable
***
*** Rust compiler 'rustc' is too new. This may or may not work.
***   Your version:     1.61.0
***   Expected version: 1.59.0
***
***
*** Rust bindings generator 'bindgen' is too new. This may or may not work.
***   Your version:     0.59.2
***   Expected version: 0.56.0
***
***
*** libclang (used by the Rust bindings generator 'bindgen') is too old.
***   Your version:    10.0.0
***   Minimum version: 11.0.0
***
make: *** [Makefile:1779: rustavailable] Error 1
```

Intalling version 11 of libclang: `sudo apt install libclang-11-dev`

Then I got:

```
make LLVM=1 rustavailable
***
*** Rust compiler 'rustc' is too new. This may or may not work.
***   Your version:     1.61.0
***   Expected version: 1.59.0
***
***
*** Rust bindings generator 'bindgen' is too new. This may or may not work.
***   Your version:     0.59.2
***   Expected version: 0.56.0
***
***
*** C compiler is too old.
***   Your Clang version:    10.0.0
***   Minimum Clang version: 11.0.0
***
Rust is available!
```

Install clang-11 and make clang point to it:

```sh
sudo apt install clang-11

sudo update-alternatives --install /usr/bin/clang clang /usr/bin/clang-11 50
update-alternatives: using /usr/bin/clang-11 to provide /usr/bin/clang (clang) in auto mode
```

https://azrael.digipen.edu/~mmead/www/mg/update-compilers/index.html

Adding `rust-src`: `rustup component add rust-src`

### Minimum rustc version

```sh
./scripts/min-tool-version.sh rustc
1.59.0
```

Checking your current version

```sh
rustc --version
rustc 1.61.0-nightly (3b348d932 2022-02-19)
```

### Setting Rust version

Update Rust if needed with the command
`rustup override set $(scripts/min-tool-version.sh rustc)`


### .config

Copy your current configuration: `cp /boot/config-$(uname -r) .config`

Use `make LLVM=1 menuconfig` to set Rust:


Searching for `Rust`: / Rust

```
 Symbol: RUST [=n]
  │ Type  : bool
  │ Defined at init/Kconfig:2056
  │   Prompt: Rust support
  │   Depends on: RUST_IS_AVAILABLE [=y] && (ARM64 || CPU_32v6 || CPU_32v6K || PPC64 && CPU_LITTLE_ENDIAN || X86_64 [=y] || RISCV) && !MODVERSIONS [=y] && !GCC_PLUGIN_RANDSTRUCT [=n]
  │   Location:
  │     Main menu
  │ (1)   -> General setup
  │ Selects: CONSTRUCTORS [=n]
```

Let's install/set the recommended versions of rustc and bindgen:

`cargo install --version 0.56.0 bindgen`

`rustup override set $(scripts/min-tool-version.sh rustc)`

Then add the source of the Rust version:

`rustup component add rust-src`

### Compiling

`make LD=/usr/bin/ld.lld-11 LLVM=1 -j$(nproc) 2> rs-errors.txt`

I got the error:

```
make[1]: *** No rule to make target 'debian/canonical-certs.pem', needed by 'certs/x509_certificate_list'.  Stop.
make[1]: *** Waiting for unfinished jobs....
make: *** [Makefile:1972: certs] Error 2
make: *** Waiting for unfinished jobs....
```

Fix here: https://askubuntu.com/questions/1329538/compiling-the-kernel-5-11-11

```
Change it to this:

CONFIG_SYSTEM_TRUSTED_KEYS=""
CONFIG_SYSTEM_REVOCATION_KEYS=""
```

I got the error below. I guess it was caused by out of memory ...<br>

```
Killed
FAILED: load BTF from vmlinux: Invalid argument
make: *** [Makefile:1225: vmlinux] Error 255
make: *** Deleting file 'vmlinux'
```

The `.config` that I copied from my machine has many drivers enabled
which might have caused this problem.

Let's try with `make LD=/usr/bin/ld.lld-11 LLVM=1 localmodconfig`, which
sets only the modules that are currently loaded.

I just kept `enter` pressed to select the default for everything ... =)

Then run `make LD=/usr/bin/ld.lld-11 LLVM=1 menuconfig` to enable Rust support.

You need to disable `Enable loadable module support` in order to enable Rust.

  - General setup -> Rust support
  - Kernel hacking -> Rust hacking
  - Sample kernel code -> Rust samples

Compiling: `make LD=/usr/bin/ld.lld-11 LLVM=1 -j7 2> rs-errors.txt`

Compilation worked. The `.config` that I used was [rust-config-ok-1](rust-config-ok-1).

### Running with Qemu

```sh
qemu-system-x86_64 -kernel arch/x86_64/boot/bzImage -initrd ramdisk.img -nographic -m 512 -append console=ttyS0
```

```
(initramfs) dmesg | grep rust
[   88.468047] process '/usr/bin/dmesg' started with executable stack
[    3.060536] Initialise system trusted keyrings
[    3.689263] rust_minimal: Rust minimal sample (init)
[    3.689780] rust_minimal: Am I built-in? true
[    3.690245] rust_print: Rust printing macros sample (init)
[    3.690639] rust_print: Emergency message (level 0) without args
[    3.690936] rust_print: Alert message (level 1) without args
[    3.691165] rust_print: Critical message (level 2) without args
[    3.691409] rust_print: Error message (level 3) without args
[    3.691660] rust_print: Warning message (level 4) without args
[    3.691917] rust_print: Notice message (level 5) without args
[    3.692183] rust_print: Info message (level 6) without args
[    3.692476] rust_print: A line that is continued without args
[    3.692777] rust_print: Emergency message (level 0) with args
[    3.693284] rust_print: Alert message (level 1) with args
[    3.693511] rust_print: Critical message (level 2) with args
[    3.693753] rust_print: Error message (level 3) with args
[    3.693975] rust_print: Warning message (level 4) with args
[    3.694195] rust_print: Notice message (level 5) with args
[    3.694630] rust_print: Info message (level 6) with args
[    3.694870] rust_print: A line that is continued with args
[    3.695189] rust_module_parameters: Rust module parameters sample (init)
[    3.698763] rust_module_parameters: Parameters:
[    3.699072] rust_module_parameters:   my_bool:    true
[    3.699326] rust_module_parameters:   my_i32:     42
[    3.699820] rust_module_parameters:   my_str:     default str val
[    3.700091] rust_module_parameters:   my_usize:   42
[    3.700335] rust_module_parameters:   my_array:   [0, 1]
[    3.700865] rust_sync: Rust synchronisation primitives sample (init)
[    3.701159] rust_sync: Value: 10
[    3.701513] rust_sync: Value: 10
[    3.701767] rust_chrdev: Rust character device sample (init)
[    3.702466] rust_miscdev: Rust miscellaneous device sample (init)
[    3.710841] rust_stack_probing: Rust stack probing sample (init)
[    3.711160] rust_stack_probing: Large array has length: 514
[    3.711497] rust_semaphore: Rust semaphore sample (init)
[    3.712233] rust_semaphore_c: Rust semaphore sample (in C, for comparison) (init)
```

### Changing the Rust code

Let's change the code to see how long compilation will take and check if it will work.

```sh
samples/rust (rust)$ grep -r "Am I" .
./rust_minimal.rs:        pr_info!("Am I built-in? {}\n", !cfg!(MODULE));
```

```diff
diff --git a/samples/rust/rust_minimal.rs b/samples/rust/rust_minimal.rs
index f0488262bcc9..beac32581443 100644
--- a/samples/rust/rust_minimal.rs
+++ b/samples/rust/rust_minimal.rs
@@ -19,7 +19,7 @@ struct RustMinimal {
 impl KernelModule for RustMinimal {
     fn init(_name: &'static CStr, _module: &'static ThisModule) -> Result<Self> {
         pr_info!("Rust minimal sample (init)\n");
-        pr_info!("Am I built-in? {}\n", !cfg!(MODULE));
+        pr_info!("Am I built-in module? {}\n", !cfg!(MODULE));

         Ok(RustMinimal {
             message: "on the heap!".try_to_owned()?,
```

The compilation is pretty fast (I guess less than 10 seconds).
Most time is spent linking and generating the image.


### Other links that might help

[lld-link: Command not found (Ubuntu 18.04)](https://github.com/XboxDev/nxdk/issues/56)

[Building Linux with Clang/LLVM](https://github.com/Rust-for-Linux/linux/blob/rust/Documentation/kbuild/llvm.rst)

[I didn't find "Rust support" in "General setup"](https://github.com/Rust-for-Linux/linux/issues/690)

[How can I downgrade or install an older version of a tool I installed with `cargo install`?](https://stackoverflow.com/questions/49639342/how-can-i-downgrade-or-install-an-older-version-of-a-tool-i-installed-with-carg)

https://tuxthink.blogspot.com/2019/09/scriptslink-vmlinuxsh-line-45-killed.html


