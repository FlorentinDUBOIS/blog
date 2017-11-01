---
title: "How use systemd-boot as bootloader"
date: 2017-11-01T18:50:35+01:00
draft: false
---

I just installed (one more time) my laptop and this time, I switch from [GRUB](https://wiki.archlinux.org/index.php/GRUB) to [systemd-boot](https://wiki.archlinux.org/index.php/systemd-boot). I prevent you that all commands that I gonna show you are used for an [Arch Linux](https://www.archlinux.org/) installation. They may work for you, but refers you to the documentation of your distribution before begin.

So, we can start now.

## Set up

The configuration of my laptop is pretty simple, there are 2 OS, one is Microsoft Windows and the other one is Arch Linux. I first installed Windows, it is easier because it creates what it need on an empty disk. After that I set up my Linux to the [installation step](https://wiki.archlinux.org/index.php/installation_guide#Boot_loader) of the bootloader. Futhermore, all are in only one solid state drive that is cut like this :

```
$ lsblk

NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda      8:0    0 931,5G  0 disk
├─sda1   8:1    0   450M  0 part
├─sda2   8:2    0   100M  0 part /boot/efi
├─sda3   8:3    0    16M  0 part
├─sda4   8:4    0   128G  0 part
├─sda5   8:5    0   600G  0 part /home
├─sda6   8:6    0     3G  0 part /boot
└─sda7   8:7    0   200G  0 part /
```

Tips: I used the `lsblk` command to display information about my SSD's partitions.

From here, we can see that Windows has 4 partitions that are `/dev/sda1`, `/dev/sda2`, `/dev/sda3` and `/dev/sda4` and Arch Linux got 3 partitions `/dev/sda5`, `/dev/sda6`, `/dev/sd7` and share one with Windows which is `/dev/sda2`.

Now, take a look of the file-system of each that is pretty important for next movements.

```
$ lsblk -fs

sda1  ntfs   Récupération <uuid>
└─sda
sda2  vfat                <uuid> /boot/efi
└─sda
sda3
└─sda
sda4  ntfs   OS 10        <uuid>
└─sda
sda5  btrfs               <uuid> /home
└─sda
sda6  btrfs               <uuid> /boot
└─sda
sda7  btrfs               <uuid> /

```
Tips: I used `lsblk` as well to show me file-system encoding with the `-fs` option.

As we can observe, all my Linux system partitions are based on the `btrfs` except the `/dev/sda2` partition that is the partition known as ESP, I will explain this later and my Windows system use `vfat` (aka `fat32`) and `ntfs`. I do not wanna that my ESP partition will be `/boot`, I prefer to see it on `/boot/efi` I thinks that is more revelant and reliable.

### ESP? What is it?

ESP is an acronym which mean `EFI System Partition` and that is on this partition we need to install the Linux bootloader to take care about others operating systems and our Linux. Furthermore, this partition __must__ be in `fat32` because your bios need to read it, in order to boot on the selected entrie which is `.efi` file.

Take a look about this partition:

```
$ tree /boot/efi -d

/boot/efi
├── EFI
│   ├── Boot
│   ├── Microsoft
│   │   ├── Boot
│   │   │   ├── fr-FR
│   │   │   ├── ... others languages directories that I omitted ...
│   │   │   └── Resources
│   │   │       └── fr-FR
│   │   └── Recovery
│   └── systemd
└── loader
    └── entries

```
Tips: I used the tree command to show you a tree of my file-system from `/boot/efi` with `-d` which mean only directories.

This is, what you got when you finished to install the bootloader, here we only saw directories.

As we can see, there is one pretty important folder which is `EFI`, on this folder there are all generated folders and files from Windows and our Linux. Furthermore, there is another folder which is `loader` that was generated when you install the systemd bootloader, we will see it later.

## Requirements

Now, we are good to start, before install the systemd bootloader on Arch Linux, we have to check if the Linux __has access to the efi variables__ and we gonna __copy all required files__ to boot.

### Check access to efi variables

So, we need to check if the folder `/sys/firmware/efi` exists, according to the [Arch Wiki](https://wiki.archlinux.org/index.php/Unified_Extensible_Firmware_Interface#Requirements_for_UEFI_variable_support). Normaly, if you had boot on your live linux on uefi mode all are good.

You may have something like this:

```
$ ls -lisrt /sys/firmware/efi

total 0
  12 0 drwxr-xr-x 2 root root    0 20 mai   15:33 efivars
6497 0 drwxr-xr-x 7 root root    0 20 mai   17:23 runtime-map
6494 0 -r--r--r-- 1 root root 4096 20 mai   17:23 runtime
6492 0 -r-------- 1 root root 4096 20 mai   17:23 systab
6495 0 -r--r--r-- 1 root root 4096 20 mai   17:23 config_table
6493 0 -r--r--r-- 1 root root 4096 20 mai   17:23 fw_vendor
6496 0 -r--r--r-- 1 root root 4096 20 mai   17:23 fw_platform_size
```
Tips: the `ls` command show you files on a directories

All are good? go to the next requirements.

Oh no! You cannot acces to this directory, you have to reboot in uefi mode.

### Copy required files

We have to copy each initramfs and vmlinuz from each kernel in order to boot. I will only copy files for the `linux` kernel here.

Commands to do:

```
$ cp /boot/vmlinuz-linux /boot/efi/vmlinuz-linux
$ cp /boot/initramfs-linux.img /boot/efi/initramfs-linux.img
$ cp /boot/initramfs-linux-fallback.img /boot/efi/initramfs-linux-fallback.img
```

## Installation

Finaly, we gotcha. Now, we had check that all requirements are satisfied, we can installed the bootloader.

```
$ bootctl --path=/boot/efi install

Copied "/usr/lib/systemd/boot/efi/systemd-bootx64.efi" to "/boot/efi/EFI/systemd/systemd-bootx64.efi".
Copied "/usr/lib/systemd/boot/efi/systemd-bootx64.efi" to "/boot/efi/EFI/BOOT/BOOTX64.EFI".
...
```
Tips: I used the `--path=/boot/efi` option because my ESP is located as `/boot/efi`, by default it wanna install on `/boot`.

We see output from the command that said to us which files were copied and a message to report to us that all are good. However, we see a message which mean that you were installed your bootloader on a no-EFI partition. You have to recheck on what partition we have to install the partiton, is it an ESP ? Recheck as well, all requirements.

### Create Linux entries

Now, we have to create entries for our Linux to boot in. Do you remenber the previous schema about the ESP partition?

```
$ tree /boot/efi -d

/boot/efi
├── EFI
│   ├── Boot
│   ├── Microsoft
│   │   ├── Boot
│   │   │   ├── fr-FR
│   │   │   ├── ... others languages directories that I omitted ...
│   │   │   └── Resources
│   │   │       └── fr-FR
│   │   └── Recovery
│   └── systemd
└── loader
    └── entries

```
Tips: I used the tree command to show you a tree of my file-system from `/boot/efi` with `-d` which mean only directories.

We need to create files with extension `.conf`, two per kernel. There are one for the casual use and the other one for the fallback use. We have to put them in the directory `/boot/efi/loader/entries`.

The casual use that I named `arch-linux.conf`:

```
title 	Arch Linux
linux 	/vmlinuz-linux
initrd	/initramfs-linux.img
options   root=UUID=<uuid> rw
```
Tips: To get the `uuid` of the root partition you can use the command `lsblk -fs`

The fallback use that I named `arch-linux-fallback.conf`:

```
title 	Arch Linux (fallback)
linux 	/vmlinuz-linux
initrd	/initramfs-linux-fallback.img
options   root=UUID=<uuid> rw

```
Tips: To get the `uuid` of the root partition you can use the command `lsblk -fs`

Files are pretty simple, we can take notice of the path of the `vmlinuz` and the `initramfs` are absolute and begin at the top of the ESP partition.

### Update bootloader configuration

If you wanna update the timeout or change the default selected entrie that I did, you have to modify the file located at `/boot/efi/loader/loader.conf`, here there is an example:

```
default arch-linux
timeout 10
```

I set a timeout of `10` seconds and my default entrie is `arch-linux`.

Now, you can reboot without worried systemd-boot will automatically find Windows and Arch Linux.

If you wanna more information, refers you to the great [Arch Wiki](https://wiki.archlinux.org/index.php/systemd-boot), it has so many secrets inside that they just ask to be discover.

## Bonus: Update on Linux kernel or systemd update

On Arch Linux, the package manager knows as `pacman` may be hooked to do a command when a package is upgraded. To upgrade our `vmlinuz`, `initiramfs` files and systemd-boot configuration we need to create 4 files on the directory `/etc/pacman.d/hooks` (you may have to create it). Each file has the following structure :

```
[Trigger]
Type = Package
Operation = Upgrade
Target = <package>

[Action]
Description = <description>
When = PostTransaction
Exec = <command>

```

So, we have to write hooks when the package manager update the following packages which are `linux` and `systemd`. Followings there are hooks for the `systemd` upgrade:

Hook that I named `systemd-boot.hook`:
```
[Trigger]
Type = Package
Operation = Upgrade
Target = systemd

[Action]
Description = Updating systemd-boot ...
When = PostTransaction
Exec = /usr/bin/bootctl --path=/boot/efi update
```
Tips: Do you remember that I have to say where is my ESP partition with `--path=/boot/efi` option?

So, we can take notice from this hook that all binaries __must__ have absolute path, it is needed. Furthermore, it looks like a `systemd` configuration file, isn't it?

Now, hooks for the `linux` package upgrade, do not forget to do the same for each Linux kernel that you had installed.

Hook that I named `vmlinuz-linux.hook`:
```
[Trigger]
Type = Package
Operation = Upgrade
Target = linux

[Action]
Description = Updating vmlinuz-linux ...
When = PostTransaction
Exec = /usr/bin/cp -f /boot/vmlinuz-linux /boot/efi/vmlinuz-linux
```

Hook that I named `initramfs-linux.hook`:
```
[Trigger]
Type = Package
Operation = Upgrade
Target = linux

[Action]
Description = Updating initramfs-linux ...
When = PostTransaction
Exec = /usr/bin/cp -f /boot/initramfs-linux.img /boot/efi/initramfs-linux.img
```

Hook that I named `initramfs-linux-fallback.hook`:
```
[Trigger]
Type = Package
Operation = Upgrade
Target = linux

[Action]
Description = Updating initramfs-linux-fallback ...
When = PostTransaction
Exec = /usr/bin/cp -f /boot/initramfs-linux-fallback.img /boot/efi/initramfs-linux-fallback.img

```

Now, we can enjoy our beautiful systemd bootloader that update automaticaly.

