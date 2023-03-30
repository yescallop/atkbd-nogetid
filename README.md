# atkbd-nogetid

A Linux kernel patch that fixes keyboard initialization on Lenovo Yoga 14s 2021 / 小新Pro 14 2021 / Yoga Slim 7 Pro / IdeaPad Slim 7 Pro (14", Intel, Tiger Lake) laptops.

This patch is only tested on Yoga 14sIHU 2021. However, it would presumably work on other machines listed below, as they share the same BIOS firmware (with version `FJCN\d\dWW`).

## Supported machines

- `82FX`: Yoga Slim 7 Pro 14ITL5
- `82G2`: Yoga 14sITL 2021
- `82GH`: Lenovo 小新Pro 14ITL 2021
- `82NC`: Yoga 14sIHU 2021 / Lenovo 小新Pro 14IHU 2021 / Yoga Slim 7 Pro 14IHU5
- `82NH`: Yoga 14sIHU 2021 O / Yoga Slim 7 Pro 14IHU5 O
- `82QT`: IdeaPad Slim 7 Pro 14IHU5 

## Installation on Arch Linux

```bash
pacman -S dkms linux-headers
git clone https://github.com/yescallop/atkbd-nogetid
dkms install atkbd-nogetid --force
mkinitcpio -P
```

Then, remove all extra kernel parameters for `i8042` or `atkbd` from the boot loader.
Reboot the laptop for the patch to take effect.

The source file `atkbd.c` is currently at [c99e3ac][1], which corresponds to kernel version `6.1` and above.
You may want to replace it with another version. It'll be automatically patched on installation.

[1]: https://github.com/torvalds/linux/blob/c99e3ac632f9dfa4e363cf370dea7467ebb0f367/drivers/input/keyboard/atkbd.c

## Problem details

A similar problem on HP Spectre x360 13-aw2xxxng had a workaround provided by Anton Zhilyaev ([forum][2], [patch][3]). This workaround, however, turns out to work only once in a while on Yoga 14sIHU 2021.

By analyzing the logs ([without patch][4], [with Anton's patch][5]), we can see that this i8042 implementation exhibits more erroneous behaviors in response to the `GETID` command than the HP one did. Not only does it sometimes deassert the interrupt after the ACK byte is read, but it also returns invalid ID bytes (containing only one byte) almost always.

As a result, it usually takes more time, or possibly forever for the keyboard to initialize on the affected machines.

The patch included in this repo makes the `atkbd_probe` function set the keyboard ID to `0xab83` without sending a `GETID` command. This should work in the most cases since the affected machines are laptops.

[2]: https://bbs.archlinux.org/viewtopic.php?pid=1953190#p1953190
[3]: https://patchwork.kernel.org/project/linux-input/patch/20210201160336.16008-1-anton@cpp.in/
[4]: https://gist.githubusercontent.com/yescallop/5a97d010f226172fafab0933ce8ea8af/raw/623871fdf233bc96a551e27111976cbc266380ea/i8042-lenovo-82nc-patch-none.log
[5]: https://gist.githubusercontent.com/yescallop/20de0b10410ec8a8c662eec7f8326569/raw/62b03331a54b3c0ba89a979a5d89db9e9e60dff5/i8042-lenovo-82nc-patch-anton.log