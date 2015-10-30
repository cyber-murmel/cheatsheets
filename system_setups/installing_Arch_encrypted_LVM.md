#  Installing Encrypted Arch Linux
## Motivation:
The idea behind Arch Linux is to use Linux learnin by doing, which is what I intend to do.
## Planned Setup:
Arch will be installed on one device \(laptop\) with one hard drive. The network access will
be via WiFi. Full disk encryption will be applied with cryptsetup \(LUKS\).
[LVM on LUKS](https://wiki.archlinux.org/index.php/Dm-crypt/Encrypting_an_entire_system#LVM_on_LUKS)

physical volume	| /dev/sda1	| /dev/sda1
----------------|---------------|----------------
content		| /boot		| LVM

## Start:

Get the image and the MD5 sums
```
$ wget http://mirror.rackspace.com/archlinux/iso/2015.10.01/archlinux-2015.10.01-dual.iso
$ wget http://mirror.rackspace.com/archlinux/iso/2015.10.01/md5sums.txt
```
Check the MD5 sum
```
$ md5sum archlinux-2015.10.01-dual.iso -c md5sums.txt
```
Flash images to medium
```
$ dd if=archlinux-2015.10.01-dual.iso | pv -b | dd of=/dev/sd*
```
Boot
Change Keyboard Layout
```
$ loadkeys de-latin1
```
Set up network
```
$ wifi-menu
```
List storage devices
```
$ lsblk
```
Wipe $device (shred with verbose output, overwrite with zeros at the end)
```
$ shred -v -z $device
```
Create physical volumes (partitions)
```
$ parted $device
(parted) mklabel msdos
(parted) mkpart primary ext4 1MiB 256MiB
(parted) set 1 boot on
(parted) mkpart primary 256MiB 100%
(parted) quit
```
Create LUKS encrypted container
```
$ cryptsetup luksFormat /dev/sda2
```
Open container as "luks"
```
$ cryptsetup open --type luks /dev/sda2 luks
```
Create physical volume
```
$ vgcreate lvm /dev/mapper/luks
```
Create logical volumes
```
$ lvcreate -L 4G lvm -n swapvol
$ lvcreate -l +100%FREE lvm -n rootvol
```
Format logical volumes
```
$ mkfs.ext4 /dev/mapper/lvm-rootvol
$ mkswap /dev/mapper/lvm-swapvol
```
Mount
```
$ mount /dev/MyStorage/rootvol /mnt
$ swapon /dev/MyStorage/swapvol
```
Format boot partition
```
$  mkfs.ext3 /dev/sda1
```
Mount boot partition
```
$ mkdir /mnt/boot
$ mount /dev/sdbY /mnt/boot
```
Select mirror by uncommenting
```
$ nano /etc/pacman.d/mirrorlist
```
Install base packages
```
$ pacstrap -i /mnt base base-devel
```
Generate fstab with universally unique identifiers
```
$ genfstab -U /mnt > /mnt/etc/fstab
```
Copy networkconfig and change root
```
$ cp /etc/netctl /mnt/etc/netctl
$ arch-chroot /mnt /bin/bash
```
Uncomment corresponding locales in /etc/locale.gen and generate locales
```
$ nano /etc/locale.gen
$ locale-gen
$ echo "LANG=en_US.UTF-8" > /etc/locale.conf
$ echo "KEYMAP=de-latin1" > /etc/vconsole.conf
```
Select time zone 
```
$ tzselect
```
Edir mkinitcpio.conf
```
$ nano /etc/mkinitcpio.conf
HOOKS="... encrypt lvm2 ... filesystems ..."
$ mkinitcpio -p linux
```
Install bootloader
```
$ pacman -S grub os-prober
$ grub-install --recheck /dev/sda
$ grub-mkconfig -o /boot/grub/grub.cfg
```
Configure Network
```
$ nano /etc/hostname
$ pacman -S iw wpa_supplicant dialog
```
Set Password and exit
```
$ passwd
$ exit
```
Unmount and reboot 
```
$ umount -R /mnt
$ reboot
```
