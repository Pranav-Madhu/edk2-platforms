Raspberry Pi 4 Platform
=======================

# Summary

This is a port of 64-bit Tiano Core UEFI firmware for the Raspberry Pi 4 platforms.

This is intended to be useful 64-bit [TF-A](https://www.trustedfirmware.org/) +
UEFI implementation for the Raspberry Pi 4 which should be good enough for most
kind of UEFI development, as well as for running consummer Operating Systems.

Raspberry Pi is a trademark of the [Raspberry Pi Foundation](https://www.raspberrypi.org).

# Status

This firmware is still in development stage, meaning that it comes with the
following __major__ limitations:

- xHCI USB may only work in pre-OS phase due to nonstandard DMA constraints.
  For 4 GB models running Linux, limiting the RAM to 3 GB should help.
  The 3 GB limitation is currently enabled by default in the user settings.
- Device Tree boot of OSes such as Linux may not work at all.
  For this reason, the provision of a Device Tree is disabled by default in
  the user settings, in order to enforce ACPI boot.
- Serial I/O from the OS may not work due to CPU throttling affecting the
  miniUART baudrate. This can be worked around by using the PL011 UART
  through the device tree overlays.

# Building

Build instructions from the top level edk2-platforms Readme.md apply.

# Booting the firmware

1. Format a uSD card as FAT16 or FAT32
2. Copy the generated `RPI_EFI.fd` firmware onto the partition
3. Download and copy the following files from https://github.com/raspberrypi/firmware/tree/master/boot
  - `bcm2711-rpi-4-b.dtb`
  - `fixup4.dat`
  - `start4.elf`
  - `overlays/miniuart-bt.dbto` or `overlays/disable-bt.dtbo` (Optional)
4. Create a `config.txt` with the following content:
    ```
    arm_64bit=1
    enable_uart=1
    enable_gic=1
    armstub=RPI_EFI.fd
    disable_commandline_tags=1
    ```
    Additionally, if you want to use PL011 instead of the miniUART, you can add the lines:
    ```
    device_tree_address=0x1f0000
    device_tree_end=0x200000
    device_tree=bcm2711-rpi-4-b.dtb
    dtoverlay=miniuart-bt
    ```
    Note that doing so requires `miniuart-bt.dbto` to have been copied into an `overlays/`
    directory on the uSD card. Alternatively, you may use `disable-bt` instead of
    `miniuart-bt` if you don't require BlueTooth.
5. Insert the uSD card and power up the Pi.

# Notes

## ARM Trusted Firmware (TF-A)

The TF-A binary was compiled from the latest TF-A release.
No aleration to the official source has been applied.

For more details on the TF-A compilation, see the relevant
[Readme](https://github.com/tianocore/edk2-non-osi/blob/master/Platform/RaspberryPi/RPi4/TrustedFirmware/Readme.md)
in the `TrustedFirmware/` directory from `edk2-non-osi`.

## Device Tree

You can pass a custom Device Tree and overlays using the following:

```
(...)
disable_commandline_tags=2
device_tree_address=0x1f0000
device_tree_end=0x200000
device_tree=bcm2711-rpi-4-b.dtb
```

Note: the address range **must** be `[0x1f0000:0x200000]`.
`dtoverlay` and `dtparam` parameters are also supported **when** providing a Device Tree`.

## Custom `bootargs`

This firmware will honor the command line passed by the GPU via `cmdline.txt`.

Note, that the ultimate contents of `/chosen/bootargs` are a combination of several pieces:
- Original `/chosen/bootargs` if using the internal DTB. Seems to be completely discarded by GPU when booting with a custom device tree.
- GPU-passed hardware configuration. This one is always present.
- Additional boot options passed via `cmdline.txt`.

# Limitations

## NVRAM

The Raspberry Pi has no NVRAM.

NVRAM is emulated, with the non-volatile store backed by the UEFI image itself. This
means that any changes made in UEFI proper are persisted, but changes made from within
an Operating System aren't.

## RTC

The Rasberry Pi has no RTC.

An `RtcEpochSeconds` NVRAM variable is used to store the boot time.
This should allow you to set whatever date/time you want using the Shell date and
time commands. While in UEFI or HLOS, the time will tick forward.
`RtcEpochSeconds` is not updated on reboots.
