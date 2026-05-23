## Introduction
One script, in-place Ubuntu installer (replace ChromeOS) for Chromebook. 

**Only x86 devices are supported at this moment.**

No USB drive, no RW_LEGACY, no WP unlocking needed! (It uses [Submarine](https://github.com/FyraLabs/submarine) as the bootloader)

All you need is:
1. [Turn on Developer mode on your Chromebook.](https://www.chromium.org/chromium-os/developer-library/guides/device/developer-mode/) (Short answer: Press [ESC + Refresh + Power button], after the recovery screen appeared, press [Ctrl + D])
2. Once ChromeOS started in developer mode, just connect to Wi-Fi. **No need** to login with Google Account!
3. Press [Ctrl+Alt+F2`(Refresh/Forward)`] enter VT2 console, login with `root` (should be no password).
4. Enter following commands:
    ````bash
    cd /tmp
    curl -LOf github.com/liyafe1997/crobuntu/raw/main/crobuntu
    bash crobuntu
    ````
5. Follow the script prompt to continue (Basically select Ubuntu version and desktop environment)
6. Wait it to be finished. After reboot, you will have Ubuntu!
7. If you want to go back to ChromeOS, press [ESC+Refresh+Power] to recover. Read [Recover your Chromebook](https://support.google.com/chromebook/answer/1080595) to learn more.

Note:
1. Default username: `ubuntu`, password: `ubuntu`.
2. Web browser might not be installed because snap pacakegs could not be installed automatically during the `chroot` setup process. You can just simply do `snap install firefox` to get Firefox installed.
3. This script will preseve the ChromeOS Cloud Recovery partition for you (aka MINIOS-B, a small parition at the end of the disk). If your device supports cloud recovery, you should be able to go back to ChromeOS just by press [ESC+Refresh+Power] and select [Recover using internet], without making a USB drive. If you think you don't need that you can also delete it and expand the ext4 rootfs in Ubuntu. Once you delete that partition, you have to use a USB drive to recover ChromeOS.

## No Sound on Ubuntu?
Check https://github.com/WeirdTreeThing/chromebook-linux-audio

## What the script does (Techical details)
It downloads `submarine-x86_64.zip` from this repository (I got it from https://nightly.link/FyraLabs/submarine/workflows/build/main/submarine-x86_64.zip, which is the offical release place of submarine), unzip it, and use `dd` to write `submarine.bin` to internal disk.

Basically `submarine` is a mini Linux kernel which can be loaded by Chromebook's bootloader `depthcharge`, it searches for something like `grub.cfg` on all partitions and parse it, then use `kexec` to execute/boot that real distro kernel.

So we need a `EFI` partition to make Ubuntu's `grub-install` happy, we don't really need EFI things but `grub-install` and `update-grub` can install grub to that (trick it we have a EFI-like installation). That's also for possible kernel upgrading compatible in the future, `apt/dpkg` will trigger `update-grub` to generate new kernel entrance in that `EFI` partition. So we should not somehow hardcode the kernel/entrance just make `submarine` works at this moment.

And then we need the third parition which is the ext4 rootfs for Ubuntu.

Then it will setup a ~10GB staging loop device at the end of the disk. The purpose is, if we just use the third ext4, it is on the beginning of the disk and it will overwrite the current ChromeOS's rootfs. Even in the script I've adready stop `system-services` (which stops most of the ChromeOS's userspace stuffs, to prevent them still read the fs randomly), it still risky to cause random reboot if some processes still read the fs but the fs is corrupted (overwrited by our ext4 fs). Overwrite the end of the disk could be much safer because generally there should be not much data on that with a fresh ChromeOS, at least, while I am testing, it never cause random reboots.

Then it will format the staging loop as ext4, download the Ubuntu's base system tar ball and unpack to the temporary staging loop, chroot, install packages, install grub to the EFI partiton, etc.

Once it finishes, exit chroot and `dd` the staging loop ext4 to the real partiton. Since the `dd` process is not so long and no any other activity, during this process, also should be safe and would not cause any reboots.

So for this ~10GB staging setup, you need at least 2x10GB ≈ 20GB+ internal disk size to make it works(able to `dd` back to the real parition without overlap). Which means, this script should work with 32GB and above Chromebooks, 16GB models would not work. If you have a small storage (16GB or less), you still can manually adjust `STAGING_SIZE_MIB` and `MIN_NON_OVERLAP_DISK_MIB` in the script to make it works. But be aware of, install desktop environment may need a lot of space, for small staging size may not enough for some large desktop setup (like `kubuntu-desktop`, it needs maybe 8GB+).

Once the `dd` is done, your ChromeOS's partitions and fs might be corrupted and destroyed and replaced with the Ubuntu's new fs. So just press `Refresh + Power Button` to perform a force reboot, then it will boot into `submarine` and your Ubuntu installation!
