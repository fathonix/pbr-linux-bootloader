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

There are two files provided for the first stage:
- plb1.s - This code looks for a GPT partition with attribute bit 2 set ("Legacy
  BIOS bootable"), or find an active MBR partition. This is recommended for
  single-boot setups.
- plb1ss.s - This code looks for the first GPT partition with valid PBR
  signature regardless whether the attribute bit 2 is set, or the first MBR
  partition with valid PBR signature regardless which one has the active flag.
  This is to work around Windows that wants its partition to be active at all
  times, or if you are unable to set the attribute bit for some reason.

There are two files provided for the second stage a.k.a. the Linux bootloader:
- plb2boot.s - This code looks for files named `vmlinuz`, `initrd.img` and
  `cmdline.txt` in the parent directory, suitable for `/boot` partition.
- plb2esp.s - This code looks for files named `vmlinuz`, `initrd.img` and
  `cmdline.txt` in the `EFI` directory, suitable for hybrid legacy and EFI boot.

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
- [doslinux](https://github.com/haileys/doslinux) for jumping between 16-bit and
  Linux mechanism.

# License

This Linux bootloader is covered under The MIT License.

The first stage chainloader is based on Apple's boot0 code, which is licensed
under APSL.
