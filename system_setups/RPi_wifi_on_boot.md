#  Configuring a Raspberry Pi (or an other Linux machine) to start WiFi on boot (with a static IP adress)
Sometimes you want to integrate you Pi into your network for automation purposes. Therefore it shall connect automaticly and a static IP makes it way easier to find.
I run Arch Linux in my Raspberry Pi, but this also should work for other linux flavours. To connect to WiFi, I use a cheap USB dongle. I recomment using an USB to TTL adapter so connect to the serial console of the Pi. Loggin in as root is advised.

## Step 1: Configuring the wpa_supplicant.conf
Firt we need the name of the interface we want to use. In my case it's `wlan0`. You can find out yours with:
```bash
# ip link
```
Next we need to create a wpa_supplicant config file for our network:
```bash
# touch /etc/wpa_supplicant/wpa_supplicant-$yourInterfaceName.conf
# wpa_passphrase "$ssidOfYourAccessPoint" "$passphraseOfYourAccessPoint" >> /etc/wpa_supplicant/wpa_supplicant-$yourInterfaceName.conf
```
You can use the last command to add more access points.
