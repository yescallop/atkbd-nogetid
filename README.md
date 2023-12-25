# atkbd-nogetid

A Linux kernel patch that fixes failed keyboard initialization on some Lenovo Yoga / XiaoXinPro / IdeaPad (14", Intel) laptops.

> [!IMPORTANT]
> The problem was resolved upstream for all laptops/portables in [936e4d4](https://github.com/torvalds/linux/commit/936e4d49ecbc8c404790504386e1422b599dec39) (kernel version `6.7` and above). Please update your kernel if possible, or use this patch at your own risk.

## Presumably supported machines

This patch is tested only on the *italicized machines*. However, it would presumably work on other machines listed below or sharing the same problem. If you find it working on yours, you can request to mark it as so or add it into the list by creating an issue.

Reports and/or illustrations (dmesg logs) of the problem are included in the footnotes.

- `82D1`: Yoga Slim 9 14ITL5[^82D1.1][^82D1.2]
- `82D2`: IdeaPad Slim 9 14ITL5[^82D2.1][^82D2.2][^82D2.3][^82D2.4]
- `82FX`: Yoga Slim 7 Pro 14ITL5[^82FX.1][^82FX.2]
- `82G2`: *Yoga 14sITL 2021*[^82G2+82NC][^82G2]
- `82GH`: Lenovo XiaoXinPro 14ITL 2021[^82GH.1][^82GH.2]
- `82NC`: *Yoga 14sIHU 2021*[^82G2+82NC] / Lenovo XiaoXinPro 14IHU 2021[^82NC.1][^82NC.2] / Yoga Slim 7 Pro 14IHU5[^82NC.3]
- `82NH`: Yoga 14sIHU 2021 O / Yoga Slim 7 Pro 14IHU5 O[^82NH]
- `82QT`: IdeaPad Slim 7 Pro 14IHU5[^82QT.1][^82QT.2][^82QT.3]
- `82TK`: Yoga Pro 14s IAH7[^82TK.1] / *Yoga Slim 7 ProX 14IAH7*[^82TK.2][^82TK.3][^82TK.4]

[^82D1.1]: https://wiki.archlinux.org/title/Lenovo_Yoga_Slim_9_(Intel)#Keyboard
[^82D1.2]: https://linux-hardware.org/?probe=be0654391c&log=dmesg
[^82D2.1]: https://www.reddit.com/r/Fedora/comments/yjn1xd/lenovo_laptop_keyboard_keys_not_working_f36/
[^82D2.2]: https://linux-hardware.org/?probe=92f4dfc282&log=dmesg
[^82D2.3]: https://linux-hardware.org/?probe=90e142cbe1&log=dmesg
[^82D2.4]: https://linux-hardware.org/?probe=a8d6ad51af&log=dmesg
[^82FX.1]: https://bbs.archlinux.org/viewtopic.php?id=273790
[^82FX.2]: http://linux-hardware.org/?probe=1b8f927465&log=dmesg
[^82G2+82NC]: https://wiki.archlinux.org/title/Lenovo_Yoga_14s_2021#Keyboard_not_functioning_properly
[^82G2]: https://bbs.archlinuxcn.org/viewtopic.php?pid=57412#p57412
[^82GH.1]: https://forum.suse.org.cn/t/topic/14779
[^82GH.2]: http://linux-hardware.org/?probe=bb63ba3801&log=dmesg
[^82NC.1]: https://zhuanlan.zhihu.com/p/428327818
[^82NC.2]: http://linux-hardware.org/?probe=bb63ba3801&log=dmesg
[^82NC.3]: https://askubuntu.com/questions/1352604/ubuntu-20-04-keyboard-not-working-on-lenovo-yoga-slim-7i-pro#comment2524117_1359483
[^82NH]: https://ubuntuforums.org/showthread.php?t=2481173
[^82QT.1]: https://discussion.fedoraproject.org/t/screen-and-keyboard-issues/76409
[^82QT.2]: https://linux-hardware.org/?probe=a6af7624cd&log=dmesg
[^82QT.3]: https://linux-hardware.org/?probe=572cb48d3c&log=dmesg
[^82TK.1]: https://zhuanlan.zhihu.com/p/583792789
[^82TK.2]: https://forums.fedoraforum.org/showthread.php?329036-Laptop-keyboard-is-not-working
[^82TK.3]: https://discussion.fedoraproject.org/t/issue-with-the-keyboard-not-working-on-touch-screen-laptop/71036
[^82TK.4]: https://bugzilla.kernel.org/show_bug.cgi?id=216994

## Installation on Arch Linux

```bash
pacman -S dkms linux-headers
git clone https://github.com/yescallop/atkbd-nogetid
dkms install atkbd-nogetid
mkinitcpio -P
```

Then, remove all extra kernel parameters for `i8042` or `atkbd` from the boot loader.
Reboot the laptop for the patch to take effect.

Note: If your machine isn't listed above, add an extra kernel parameter `atkbd.nogetid` in the boot loader.

The source file `atkbd.c` is currently at [c4c7eac][1], which corresponds to kernel version `6.5` and above.
You may want to replace it with another version. It'll be automatically patched on installation.

[1]: https://github.com/torvalds/linux/blob/c4c7eac8ee78d896635ce05d7a1c3f813fcbe24c/drivers/input/keyboard/atkbd.c

## Problem details

A similar problem on HP Spectre x360 13-aw2xxxng had a workaround provided by Anton Zhilyaev ([forum][2], [patch][3]). This workaround, however, turns out to work only once in a while on Yoga 14sIHU 2021.

By analyzing the logs ([without patch][4], [with Anton's patch][5]), we can see that this i8042 implementation exhibits more erroneous behaviors in response to the `GETID` command than the HP one did. Not only does it sometimes fail to raise interrupts for some response bytes, but it also returns invalid ID bytes (containing only one byte) almost always (which is why Anton's patch doesn't really work).

The patch included in this repo adds quirks in `atkbd` that skips the `GETID` command on the affected machines. This should work in the most cases since these machines are laptops.

[2]: https://bbs.archlinux.org/viewtopic.php?pid=1953190#p1953190
[3]: https://patchwork.kernel.org/project/linux-input/patch/20210201160336.16008-1-anton@cpp.in/
[4]: https://gist.github.com/yescallop/5a97d010f226172fafab0933ce8ea8af
[5]: https://gist.github.com/yescallop/20de0b10410ec8a8c662eec7f8326569
