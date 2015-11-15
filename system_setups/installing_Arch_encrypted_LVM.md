#  Installing Encrypted Arch Linux
## Motivation:
The idea behind Arch Linux is to use Linux learnin by doing, which is what I intend to do.
## Planned Setup:
Arch will be installed on one device \(laptop\) with one hard drive. The network access will
be via WiFi. Full disk encryption will be applied with cryptsetup \(LUKS\).
[LVM on LUKS](https://wiki.archlinux.org/index.php/Dm-crypt/Encrypting_an_entire_system#LVM_on_LUKS)

physical volume | /dev/sda1     | /dev/sda1
----------------|---------------|----------------
content         | /boot         | LVM

## Start:
On your machine:

```bash
$ wget http://mirror.rackspace.com/archlinux/iso/2015.10.01/archlinux-2015.10.01-dual.iso #get the image
$ wget http://mirror.rackspace.com/archlinux/iso/2015.10.01/md5sums.txt #get the MD5 sums
$ md5sum archlinux-2015.10.01-dual.iso -c md5sums.txt # check for integrity
$ dd if=archlinux-2015.10.01-dual.iso | pv -b | dd of=/dev/sd* # flash to storage device
```
Boot
``` bash
$loadkeys de-latin1                             # change keyboard layout
$ wifi-menu                                     # setup network
$ lsblk                                         # list storage devices
$ shred -v -z /dev/sda                          # shred $device, overwrite with zeros
$ parted $device                                # create partitions
(parted) mklabel msdos
(parted) mkpart primary ext3 1MiB 256MiB
(parted) set 1 boot on
(parted) mkpart primary 256MiB 100%
(parted) quit
$ cryptsetup luksFormat /dev/sda2               # create crypted contaienr
$ cryptsetup open --type luks /dev/sda2 luks    # open container als "luks"
$ vgcreate volgro /dev/mapper/luks              # create volume group
$ lvcreate -L 4G volgro -n swapvol              # create swap
$ lvcreate -l +100%FREE volgro -n rootvol       # create root
$ mkfs.ext4 /dev/volgro/rootvol                 # create file system
$ mkswap /dev/volgro/swapvol                    # create file system
$ mount /dev/volgro/rootvol /mnt                # mount root
$ swapon /dev/volgro/swapvol                    # enable swap
$ mkfs.ext3 /dev/sda1                           # format boot partition
$ mkdir /mnt/boot                               # create boot directory
$ mount /dev/sda1 /mnt/boot                     # mount boot
$ nano /etc/pacman.d/mirrorlist                 # select mirror by uncommenting
$ pacstrap -i /mnt base base-devel              # install base packages
$ genfstab -U /mnt > /mnt/etc/fstab             # generate file system table with universally unique indentifiers
$ cp /etc/netctl /mnt/etc/netctl                # copy network configuration
$ arch-chroot /mnt /bin/bash                    # change root into the new system

$ nano /etc/locale.gen                          # uncomment correspondign locales
$ locale-gen                                    # generate locales
$ echo "LANG=en_US.UTF-8" > /etc/locale.conf    # set language
$ echo "KEYMAP=de-latin1" > /etc/vconsole.conf  # set keyboard layout
$ tzselect                                      # select time zone
$ nano /etc/mkinitcpio.conf                     # edit mkinitcpio.conf
HOOKS="... encrypt lvm2 ... filesystems ..."
$ mkinitcpio -p linux                           # generate initramfs
$ pacman -S grub os-prober                      # install bootloader
$ grub-install --recheck /dev/sda               # apply bootloader
$ grub-mkconfig -o /boot/grub/grub.cfg          # configure boot loader
$ nano /etc/hostname                            # set hostname
$ pacman -S iw wpa_supplicant dialog            # install netowrk config toosl
$ passwd                                        # set password
$ exit                                          # exit system
$ umount -R /mnt                                # unmount everything
$ reboot                                        # reboot
```
