
# Good First Issues

The Linux Kernel doesn't have "good first issues" like you can
find in Github projects (not that I'm aware of), but there are
some things we can look for for easy patches.

## TODOs

Many developers write `TODO` in the code, so look for them.

Of course `TODO` does not mean easy, but it's a good starting point.

## Linux Staging Tree


>The Linux Staging tree (or just "staging" from now on) is used to hold
>stand-alone[1] drivers and filesystems that are not ready to be merged into
>the main portion of the Linux kernel tree at this point in time for various
>technical reasons.

-- Greg KH

Read more here: https://lwn.net/Articles/324279/


## Look the Kernel log using `dmesg`

The errors are (usually) in red.

eg:

<div>
[    0.653702] Key type encrypted registered<br>
[    0.654413] AppArmor: AppArmor sha1 policy hashing enabled<br>
[    0.655910] integrity: Loading X.509 certificate: UEFI:db<br>
[    0.656597] integrity: <span style="color: red">Problem loading X.509 certificate -65</span><br>
</div>

Let's find where this 'Problem' message came from:

```sh
for f in $(find . | grep integrity); do grep Problem $f; done
grep: ./security/integrity: Is a directory
grep: ./security/integrity/ima: Is a directory
grep: ./security/integrity/platform_certs: Is a directory
Binary file ./security/integrity/digsig.o matches
		pr_err("Problem loading X.509 certificate %d\n", rc);
grep: ./security/integrity/evm: Is a directory
```

## Search for regressions

https://linux-regtracking.leemhuis.info/regzbot/mainline/


