ZFSBootMenu is a Dracut module that intends to provide Linux distributions with
an experience similar to FreeBSD's bootloader. By taking advantage of ZFS
features, it allows a user to have multiple "boot environments" (with different
distros, for example), manipulate snapshots before booting, and, for the
adventurous user, even bootstrap a system installation via `zfs recv`.

## Demonstration

[![asciicast](https://asciinema.org/a/FN4gWtVUPPXzgZPrCjd8mK6Vz.svg)](https://asciinema.org/a/FN4gWtVUPPXzgZPrCjd8mK6Vz)

## Key Features

* Allows selection of specific kernels among boot environments distributed
  across any number of pools.

* Facilitates cloning (or independently duplicating) prior snapshots of any
  boot environment and booting the clone.

* Works with native ZFS encryption and LUKS.

* Capable of producing UEFI bundles or separate kernel and initramfs images.

* Provides a mechanism to adjust the default boot environment for any pool.

* Allows kernel versions to be "pinned" for each boot environment, overriding
  the default behavior of booting the kernel with the highest version.

* Behavior is adjustable through (mostly) inheritable ZFS properties.

## Overview of the Boot Process

* Via GRUB, direct EFI booting, gummiboot, rEFInd or other UEFI boot manager,
  boot a stock Linux kernel along with an initramfs containing the ZFSBootMenu
  dracut module.
* Import all importable pools.
* Try to identify a default boot environment.
    - If ZFSBootMenu was started with a `root=zfsbootmenu:POOL=<pool>` argument,
      the default boot environment will be that specified by  the `bootfs`
      property on the specified pool.
    - If no preferred pool was identified, look for the first identified pool
      with a `bootfs` property to determine the default boot environment.
* If a default boot environment was identified, present a countdown timer for
  automatic boot.
* If no default environment can be found or the countdown is interrupted,
  provide an interactive boot menu.
    - Identify all potential boot environments as filesystems that mount to `/`
      or have a `legacy` mountpoint with an opt-in property set.
    - Enumerate kernels and initramfs images in the `/boot` subdirectory of
      each environment.
    - Allow selection of a desired kernel within any boot environment.
* After a boot environment is selected for boot (automatically or manually),
  load the selected kernel and initramfs, unmount all ZFS filesystems, and boot
  the boot the system.

At this point, you'll be booting into your usual OS-managed kernel and
initramfs, along with any arguments needed to correctly boot your system.

## Encryption Support

ZFSBootMenu was designed around native ZFS encryption. ZFSBootMenu will prompt
for passphrases as necessary to unlock pools or filesystems needing to support
user interaction or the standard boot process.

ZFSBootMenu also works with the standard LUKS dracut module to allow booting
from ZFS pools on a LUKS encrypted device.

## Safe by Design

Pools are always imported read-only by default. Certain pool, filesystem or
snapshot operations supported by ZFSBootMenu (but only performed by user
request) require that pools be imported read-write. If the reimport cannot be
done safely, those operations will not be permitted.

## More Details

Please refer to the [ZFSBootMenu project web site](https://github.com/zbm-dev/zfsbootmenu)
for detailed documentation.
