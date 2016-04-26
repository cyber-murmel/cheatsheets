#  Configuring a Raspberry Pi (or an other Linux machine) to start WiFi on boot (with a static IP adress)
Sometimes you want to integrate you Pi into your network for automation purposes. Therefore it shall connect automaticly and a static IP makes it way easier to find.
I run Arch Linux in my Raspberry Pi, but this also should work for other linux flavours. To connect to WiFi, I use a cheap USB dongle. I recomment using an USB to TTL adapter so connect to the serial console of the Pi. Loggin in as root is advised.

## Step 1: Configuring the wpa_supplicant.conf
Firt we need the name of the interface we want to use. In my case it's `wlan0`. You can find out yours with:
```bash
# ip link
```
Next we need to create a wpa_supplicant config file for our network. The first line makes the running supplicant available to other programs.
```bash
# echo ctrl_interface=/var/run/wpa_supplicant >> /etc/wpa_supplicant/wpa_supplicant-$yourInterfaceName.conf
# wpa_passphrase "$ssidOfYourAccessPoint" "$passphraseOfYourAccessPoint" >> /etc/wpa_supplicant/wpa_supplicant-$yourInterfaceName.conf
```
You can use the last command to add more access points.

## Step 2: Configuring the dhcpcd
Now we add some lines to the top of dhcpcd.conf that tell dhcpcd how to call wpa_supllicant and behave.
```bash
# vim /etc/dhcpcd.conf
 env wpa_supplicant_driver=wext                 # this is only needed, if your interface doesn't support the default driver
 env ifwireless=1                               # this tells dhcpcd that the interface is wireless
 interface wlan0                                # the rest is a standard dhcpcd conf
 static ip_address=.../...
 static routers=...
 static domain_name_servers=...
 ...
```
## Step 2: Configuring systemd
In the end, all we need to do, is create a dhcpcd hook by making a symlink and tell systemd to enable it.
```bash
# ln -s /usr/share/dhcpcd/hooks/10-wpa_supplicant /usr/lib/dhcpcd/dhcpcd-hooks/
# systemctl enable dhcpcd@wlan0
```

And we are done.
