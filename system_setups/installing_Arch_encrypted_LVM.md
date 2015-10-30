#  Installing Encrypted Arch Linux
## Motivation:
The idea behind Arch Linux is to use Linux learnin by doing, which is what I intend to do.
## Planned Setup:
Arch will be installed on one device \(laptop\) with one hard drive. The network access will
be via WiFi. Full disk encryption will be applied with cryptsetup \(LUKS\).
[LUKS on LVM](https://wiki.archlinux.org/index.php/Dm-crypt/Encrypting_an_entire_system#LUKS_on_LVM)

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
Create physical volume for LVM
```
$ lvm pvcreate /dev/sda2
```
Create volume group calles "lvm"
```
$ lvm vgcreate lvm /dev/sda2
```
Create logical volume in volume group
Swap
```
$ lvm lvcreate -L 2G -n swap lvm
```
