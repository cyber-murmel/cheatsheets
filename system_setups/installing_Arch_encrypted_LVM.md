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

```
$ wget http://mirror.rackspace.com/archlinux/iso/2015.10.01/archlinux-2015.10.01-dual.iso #get the image
$ wget http://mirror.rackspace.com/archlinux/iso/2015.10.01/md5sums.txt #get the MD5 sums
$ md5sum archlinux-2015.10.01-dual.iso -c md5sums.txt # check for integrity
$ dd if=archlinux-2015.10.01-dual.iso | pv -b | dd of=/dev/sd* # flash to storage device
```
Boot
```
$ loadkeys de-latin1                            # change keyboard layout
$ iw dev                                        # show network interface devices
$ ip link set $interface up                        # activate wifi device
$ wpa_supplicant -D nl80211,wext -i $interface -c <(wpa_passphrase $your_SSID $your_PSK)  # connect to WPA secured AP
$ dhclient $interface
$ lsblk                                         # list storage devices
$ fdisk $device                                # create partitions
delete
new
primary
1
<enter>
+256M
new
primary
1
<enter>
<enter>
print
write
$ cryptsetup luksFormat /dev/$device2               # create crypted contaienr
$ cryptsetup open --type luks /dev/sda2 luks    # open container als "luks"
$ volgroname=volgro_$month_$year
$ vgcreatevolgroname /dev/mapper/luks              # create volume group
##$ lvcreate -L 4G volgroname -n swapvol              # create swap
$ lvcreate -l +100%FREE volgroname -n rootvol       # create root
$ mkfs.ext4 /dev/volgroname/rootvol                 # create file system
##$ mkswap /dev/volgroname/swapvol                    # create file system
$ mount /dev/volgroname/rootvol /mnt                # mount root
##$ swapon /dev/volgroname/swapvol                    # enable swap
$ mkfs.ext4 /dev/sda1                           # format boot partition
$ mkdir /mnt/boot                               # create boot directory
$ mount /dev/$device1 /mnt/boot                     # mount boot
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
HOOKS="... udev... encrypt lvm2 (resume)... filesystems ..."
$ mkinitcpio -p linux                           # generate initramfs
$ pacman -S grub os-prober                      # install bootloader
$ grub-install --recheck /dev/$device               # apply bootloader
$ grub-mkconfig -o /boot/grub/grub.cfg          # create bootloader config
$ nano /boot/grub/grub.cfg                      # configure kernel parameters
linux ... cryptdevice=<path to encrypted blockdevice>:luks (resume=<path to swap mountpoint>) root= ... rw quiet
$ nano /etc/hostname                            # set hostname
$ pacman -S wpa_supplicant dialog               # install netowrk config tools
$ pacman -S sudo                                # install sudo
$ visudo                                        # uncomment "%sudo   ALL=(ALL) ALL"
$ useradd -m -U $username -G sudo              # add your account
$ passwd $username                                  # set user password
$ passwd                                        # set root password
$ nanp /etc/pacman.conf                         # activate multilib support by uncommenting
[multilib]
Include = /etc/pacman.d/mirrorlist
$ pacman -Syu                                   # upgrade system
$ exit                                          # exit system
$ umount -R /mnt                                # unmount everything
$ reboot                                        # reboot
```
