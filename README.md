# pbr-linux-bootloader

A single sector, x86 Linux bootloader that resides in the PBR/VBR of a FAT32
partition. This bootloader expects to find a Linux kernel with bzImage format,
an initrd/initramfs and a plain ASCII text file containing the kernel's
command-line parameters in the filesystem. This is useless if you do not plan to
use the legacy booting scheme.

# Features

- Supports both MBR partition table and GPT, searches the latter first then the
  former. Suitable for booting GPT disks on legacy systems without modifying the
  hybrid/protective MBR partition table.
- Very minimal, fits in the standard boot sector.
- Easy to change the kernel, initrd and parameters because they are kept in a
  usual FAT32 filesystem, no need for a custom MBR or GPT scheme.

# Installing

IBM PC compatible boot process, now known for legacy boot, is divided into two
parts. The first stage is BIOS locating and loading the MBR, then the second one
is MBR locating and loading the PBR. Most MBR code out there simply looks for
the active partition on the partition table and thinks that the PBR should be
there, but this is not enough with the popularity of GPT and UEFI slowly
replacing the mature but limited legacy boot and MBR. Therefore, this project also provides
the MBR code that also works with GPT to get the best of both worlds.

There are two options provided for the first stage:
- plb1 - This code looks for a GPT partition with attribute bit 2 set ("Legacy
  BIOS bootable"), or find an active MBR partition. This is recommended for
  single-boot setups.
- plb1ss - This code looks for the first GPT partition with valid PBR
  signature regardless whether the attribute bit 2 is set, or the first MBR
  partition with valid PBR signature regardless which one has the active flag.
  This is to work around Windows that wants its partition to be active at all
  times, or if you are unable to set the attribute bit for some reason.

There are two options provided for the second stage a.k.a. the Linux bootloader:
- plb2boot - This code looks for files named `vmlinuz`, `initrd.img` and
  `cmdline.txt` in the parent directory, suitable for `/boot` partition.
- plb2esp - This code looks for files named `vmlinuz`, `initrd.img` and
  `cmdline.txt` in the `EFI` directory, suitable for hybrid legacy and EFI boot.

### Automatic Install

Make sure you have `make`, `nasm`, `dd` and root access. Run this command to
install:
```bash
sudo make DEST=/dev/sdXY stage1 stage2
```
Replace `X` with the target drive letter, `Y` with the target partition number,
`stage1` with the first stage option you choose and `stage2` with one of the
second stage options. This assumes your host is Linux and the target drive has
sane MBR or GPT scheme, otherwise remove the `DEST` argument for manual install.

### Manual Install

Make sure you have `make` and `nasm`. Run this command to compile:
```bash
make stage1 stage2
```
Replace `stage1` with the first stage option you choose and `stage2` with one of
the second stage options.

On Windows, run BOOTICE as administrator then select the target drive under the
Destination Disk section, select Process MBR then Restore MBR, select
`stage1.bin` under this project folder where `stage1` is the first stage option
you have previously chosen. Once finished, go back to the BOOTICE's main window
and select Process PBR, select the target partition under the Destination
Partition section, select Restore PBR, select `stage2.bin` under this project
folder where `stage2` is the second stage option you have previously chosen.

# Technical Information

This bootloader expects that the MBR or another bootloader that chainloads this
left the drive number in `DL`.

# Inspired By

- [tiny-linux-bootloader](https://github.com/owenson/tiny-linux-bootloader)
  as the main inspiration.
- boot0* and boot1f32* code used by Clover and OpenCore to chainload DUET.
- [Syslinux](https://www.syslinux.org) and
  [ArchWiki](https://wiki.archlinux.org/title/Syslinux#GUID_partition_table)
  for making me learn that GPT attribute bit 2 is a thing.
- [doslinux](https://github.com/haileys/doslinux) as another reference for
  jumping between 16-bit and Linux mechanism.

# License

This Linux bootloader is covered under The MIT License.

The first stage chainloader is based on Apple's boot0 code, which is licensed
under APSL.
