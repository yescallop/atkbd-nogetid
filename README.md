# atkbd-nogetid

A Linux kernel patch that fixes keyboard initialization on Lenovo Yoga 14s 2021 / IdeaPad Slim 7 Pro (Intel) laptops.

## Install (DKMS)

```bash
git clone https://github.com/yescallop/atkbd-nogetid
sudo dkms install atkbd-nogetid --force
sudo mkinitcpio -P
```

Then, make sure all extra kernel parameters for `i8042` or `atkbd` are removed.
Reboot the laptop for the patch to take effect.

The source file `atkbd.c` is currently at [c99e3ac][1].
You may replace it with another version. It'll be automatically patched on installation.

[1]: https://github.com/torvalds/linux/blob/c99e3ac632f9dfa4e363cf370dea7467ebb0f367/drivers/input/keyboard/atkbd.c
