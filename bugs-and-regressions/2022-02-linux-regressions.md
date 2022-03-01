# Some Linux Kernel Regressions


```
From	Linus Torvalds <>
Date	Sun, 27 Feb 2022 15:00:26 -0800
Subject	Linux 5.17-rc6
https://lkml.org/lkml/2022/2/27/350

...

While things look reasonably normal, we _are_ getting pretty late in
the release, and we still have a number of known regressions. They
don't seem all that big and scary, but some of them were reported
right after the rc1 release, so they are getting a bit long in the
tooth. I'd hate to have to delay 5.17 just because of them, and I'm
starting to be a bit worried here. I think all the affected
maintainers know who they are...

So if you are a subsystem maintainer, and you have one of those
regressions on your list, please go make them a priority. And if you
don't know what I'm talking about, please do look up the reports by
regzbot and Guenter Roeck. I added links below to make it really easy.

But on the whole things look fine. Just a few remaining warts is all.
But the more testing to verify, the better.

               Linus

https://lore.kernel.org/all/164598944963.346832.10219407090470852309@leemhuis.info/
https://lore.kernel.org/all/20220221024743.GA4097766@roeck-us.net/
```

## From: "Regzbot (on behalf of Thorsten Leemhuis)" 

https://lore.kernel.org/all/164598944963.346832.10219407090470852309@leemhuis.info/

```
Hi Linus, Thorsten here, with a quick preface before the latest report
from regzbot.

From my side things seem to look normal right now; there are
two bluetooth regressions in mainline currently (and two others in older
releases; for one there is already a fix available), but people are
working on it. I'm aware of two other regressions in mainline, but one
is pretty special and for the other a fix hopefully will be sent on the
way soon.

If you have a minute: there is still the CIFS issue that was introduced
in a earlier cycle where I'm unsure how to handle it. For a recent
write-up of the current situation and the latest mail in the thread see:

https://lore.kernel.org/lkml/49f945f4-a8ea-825b-8c45-64c8a767631e@leemhuis.info/
https://lore.kernel.org/lkml/CAJjP=Bus1_ce4vbHXpiou1WrSe8a61U1NzGm4XvN5fYCPGNikA@mail.gmail.com/

Ciao, Thorsten

--

Hi, this is regzbot, the Linux kernel regression tracking bot.

Currently I'm aware of 24 regressions in linux-mainline. Find the
current status below and the latest on the web:

https://linux-regtracking.leemhuis.info/regzbot/mainline/

Bye bye, hope to see you soon for the next report.
   Regzbot (on behalf of Thorsten Leemhuis)
```

1) Since commit e8907f76544ffe225ab95d70f7313267b1d0c76d bluetooth scanning stopped working on my system

https://patchwork.kernel.org/project/bluetooth/patch/20220228173918.524733-1-brian.gix@intel.com/

https://linux-regtracking.leemhuis.info/regzbot/regression/f648f2e11bb3c2974c32e605a85ac3a9fac944f1.camel@redhat.com/


Fixed in: https://patchwork.kernel.org/project/bluetooth/patch/20220228173918.524733-1-brian.gix@intel.com/


Searching log using `journalctl -u bluetooth -b0`

```
-- Logs begin at Tue 2021-05-18 10:13:08 -03, end at Tue 2022-03-01 16:02:10 -03. --
mar 01 09:13:37 lesco-Lenovo-IdeaPad-S145-15API systemd[1]: Starting Bluetooth service...
mar 01 09:13:37 lesco-Lenovo-IdeaPad-S145-15API bluetoothd[757]: Bluetooth daemon 5.53
mar 01 09:13:38 lesco-Lenovo-IdeaPad-S145-15API systemd[1]: Started Bluetooth service.
mar 01 09:13:38 lesco-Lenovo-IdeaPad-S145-15API bluetoothd[757]: Starting SDP server
mar 01 09:13:38 lesco-Lenovo-IdeaPad-S145-15API bluetoothd[757]: Bluetooth management interface 1.20 initialized
mar 01 09:13:38 lesco-Lenovo-IdeaPad-S145-15API bluetoothd[757]: Failed to set mode: Blocked through rfkill (0x12)
mar 01 09:13:42 lesco-Lenovo-IdeaPad-S145-15API bluetoothd[757]: Failed to set mode: Blocked through rfkill (0x12)
mar 01 09:13:49 lesco-Lenovo-IdeaPad-S145-15API bluetoothd[757]: Endpoint registered: sender=:1.72 path=/MediaEndpoint/A2DPSink/sbc
mar 01 09:13:49 lesco-Lenovo-IdeaPad-S145-15API bluetoothd[757]: Endpoint registered: sender=:1.72 path=/MediaEndpoint/A2DPSource/sbc
mar 01 13:42:01 lesco-Lenovo-IdeaPad-S145-15API bluetoothd[757]: Endpoint unregistered: sender=:1.72 path=/MediaEndpoint/A2DPSink/sbc
mar 01 13:42:01 lesco-Lenovo-IdeaPad-S145-15API bluetoothd[757]: Endpoint unregistered: sender=:1.72 path=/MediaEndpoint/A2DPSource/sbc
mar 01 13:42:02 lesco-Lenovo-IdeaPad-S145-15API bluetoothd[757]: Failed to set mode: Blocked through rfkill (0x12)
mar 01 13:42:02 lesco-Lenovo-IdeaPad-S145-15API bluetoothd[757]: Endpoint registered: sender=:1.72 path=/MediaEndpoint/A2DPSink/sbc
mar 01 13:42:02 lesco-Lenovo-IdeaPad-S145-15API bluetoothd[757]: Endpoint registered: sender=:1.72 path=/MediaEndpoint/A2DPSource/sbc
mar 01 13:42:03 lesco-Lenovo-IdeaPad-S145-15API bluetoothd[757]: Failed to set mode: Blocked through rfkill (0x12)
```

From man page:

>journalctl may be used to query the contents of the systemd(1) journal as written by systemd-journald.service

>dmesg is used to examine or control the kernel ring buffer.
>
>The default action is to display all messages from the kernel ring buffer.


2) [PATCH] PCI: xgene: Fix IB window setup

https://lore.kernel.org/stable/20211129173637.303201-1-robh@kernel.org/

