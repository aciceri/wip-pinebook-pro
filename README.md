WIP stuff to get started on the pinebook pro.

## Using in your configuration

Clone this repository somwhere, and in your configuration.nix

```
{
  imports = [
    .../pinebook-pro/pinebook_pro.nix
  ];
}
```

That entry point will try to stay unopinionated, while maximizing the hardware
compatibility.

## Compatibility

### Tested

 * X11 with modesetting
 * Wi-Fi
 * Brightness controls

### Untested

 * Bluetooth

### Known issues

 * Suspend (or resume) fails.

### Tips

The backlight can be controlled using `light` (`programs.light.enable`).

## Image build

```
$ ./build.sh
$ lsblk /dev/mmcblk0 && sudo dd if=$(echo result/sd-image/*.img) of=/dev/mmcblk0 bs=8M oflag=direct status=progress
```

The `build.sh` script transmits parameters to `nix-build`, so e.g. `-j0` can
be used.

Once built, this image is self-sufficient, meaning that it should already be
booting, no need burn u-boot to it.

The required modules (and maybe a bit more) are present in stage-1 so the
display should start early enough in the boot process.

The LED should start up "amber, red, amber, green, amber" with this u-boot
configuration, as a way to show activity early. The kernel should set it to
green as soon as it can.

## Note about cross-compilation

This will automatically detect the need for cross-compiling or not.

When cross-compiled, all caveats apply. Here this mainly means that the kernel
will need to be re-compiled on the device on the first nixos-rebuild switch,
while most other packages can be fetched from the cache.

## `u-boot`

Assuming `/dev/mmcblk0` is an SD card.

```
$ nix-build -A pkgs.uBootPinebookPro
$ lsblk /dev/mmcblk0 && sudo dd if=result/idbloader.img of=/dev/mmcblk0 bs=512 seek=64 oflag=direct,sync && sudo dd if=result/u-boot.itb of=/dev/mmcblk0 bs=512 seek=16384 oflag=direct,sync
```

The eMMC has to be zeroed (in the relevant sectors) or else the RK3399 will use
the eMMC as a boot device first.

Alternatively, this u-boot can be installed to the eMMC.

Installing to SPI has yet to be investigated.

### Updating eMMC u-boot from NixOS

**Caution:** this could render your system unbootable. Do this when you are in
a situation where you can debug and fix the system if this happens. With this
said, it should be safe enough.

```
$ nix-build -A pkgs.uBootPinebookPro
$ lsblk /dev/disk/by-path/platform-fe330000.sdhci && sudo dd if=result/idbloader.img of=/dev/disk/by-path/platform-fe330000.sdhci bs=512 seek=64 oflag=direct,sync && sudo dd if=result/u-boot.itb of=/dev/disk/by-path/platform-fe330000.sdhci bs=512 seek=16384 oflag=direct,sync
```

## Keyboard firmware

```
 $ nix-build -A pkgs.pinebookpro-keyboard-updater
 $ sudo ./result/bin/updater step-1
 $ sudo reboot
 # ...
 $ sudo ./result/bin/updater step-2
 $ sudo reboot
```
