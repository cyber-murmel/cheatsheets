#  Installing awesome window manager on Arch:
https://wiki.archlinux.org/index.php/Awesome
http://awesome.naquadah.org/wiki/Awesome_3_configuration
```bash
$ sudo pacman -S awesome                            # install awesome
$ nano ~/.xinitrc                                   # add exec awesome
$ mkdir -p ~/.config/awesome/                       # create config folder
$ cp /etc/xdg/awesome/rc.lua ~/.config/awesome/     # copy config template
```
