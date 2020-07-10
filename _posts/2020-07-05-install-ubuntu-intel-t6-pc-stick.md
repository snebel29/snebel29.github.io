---
layout: post
title: Wifi not working in Ubuntu 20.04 LTS on Intel T6 stick computer
description: Wifi not working in Ubuntu 20.04 LTS on Intel T6 stick computer
---

I just bought one [T6 fainless computer stick from intel](https://www.amazon.co.uk/Fanless-Windows-T6-Computer-Bluetooth/dp/B07RJMFFY1?th=1) to use in my living room tv set, I ran a usb key with a live ubuntu distribution and noticed the wifi was not working at all.

To get it working I just had to copy the following firmware file `C:\Windows\Sytem32\drivers\4345r6nvram.txt` from the original Windows setup to `/lib/firmware/brcm/brcmfmac43455-sdio.txt` in the linux system, then reload the broadcom module in the kernel.

> Note the file name changed too.

```
$ sudo modprobe -r brcmfmac
$ sudo modprobe brcmfmac
```

And that's it, wifi was working perfectly after that! As a reference here is the [brcmfmac43455-sdio.txt]({{site.url}}/download/brcmfmac43455-sdio.txt) file.

