# Software I want/need on almost every system
- htop
- openssh
- rfkill
- arandr
- terminology
- git
- alsa-utils
- i3lock
- keepassx
- xorg-server
- xorg-xinit
- xorg-xinput
- xf86-input-synaptics
- awesome
- compton
- thunderbird
- screen
- sudo
- zsh
- grml-zsh-config

## infinality-bundle for nice looking fonts
add
```
[infinality-bundle]
Server = http://bohoomil.com/repo/$arch

[infinality-bundle-multilib]
Server = http://bohoomil.com/repo/multilib/$arch

[infinality-bundle-fonts]
Server = http://bohoomil.com/repo/fonts
```
to `/etc/pacman.conf`

Then import and sign the key
```
pacman-key -r 962DDE58
pacman-key --lsign-key 962DDE58
```
then install software
```
pacman -Syu
pacman -S infinality-bundle
```
