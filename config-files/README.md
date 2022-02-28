There are lots and lots of configurations available ...

With time we learn about them. =)

Use `make menuconfig` so you can browse them.
You can use '?' if you are not sure what that option does.

The more configurations you set, the longer the compilation will take.

There two easy and fast ways to generate a `.config` that will work
for your machine:
  1. take your current config: `cp /boot/config-$(uname -r) .config`
  2. Create a config based on current config and loaded modules: `make localmodconfig`


## References

https://www.kernel.org/doc/Documentation/admin-guide/README.rst

