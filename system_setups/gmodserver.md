# How to install Garry's Mod dedicated server on a Linux machine:
[Steam CMD](https://developer.valvesoftware.com/wiki/SteamCMD#32-bit_libraries_on_64-bit_Linux_systems) </br>
[Linux dedicated server](http://wiki.garrysmod.com/page/Linux_Dedicated_Server_Hosting)
```bash
$ nano /etc/pacman.conf                                                 # uncomment [multilib] and Include = /etc/pacman.d/mirrorlist
$ pacman -S lib32-gcc-libs                                              # install the dependencies

$ useradd -m steam                                                      # add new user and create home directory
$ passwd steam                                                          # set unix password for user
$ su - steam                                                            # login as steam
$ mkdir bin; cd bin                                                     # make bin directory and change into it
$ wget http://media.steampowered.com/client/steamcmd_linux.tar.gz       # get steamcmd
$ tar -xvzf steamcmd_linux.tar.gz                                       # unpack steamcmd
$ ./steamcmd.sh +login anonymous +quit                                  # first time execution

$ cd ~                                                                  # change into home directory
$ nano update_gmod.sh                                                   # create update script
# !/bin/bash

# A convenience function, to save us some work
update_server() {
	# Read the app id and the directory into a variable

	APP_ID=$1
	DIR=$2

	# Create the directory ( if it does not exist already )
	if [ ! -d "$HOME/$DIR" ]; then
		mkdir -p "$HOME/$DIR"
	fi

	# Uh-oh, it looks like we still have no directory. Report an error.
	if [ ! -d "$HOME/$DIR" ]; then
		# Describe what went wrong
		echo "ERROR! Cannot create directory $HOME/$DIR!"

		# Exit with status code 1 ( which indicates an error )
		exit 1
	fi

	# Call SteamCMD with the app ID we provided and tell it to install
	./bin/steamcmd.sh +login anonymous +force_install_dir "$HOME/$DIR" +app_update $APP_ID validate +quit
}

# Now the script actually runs update_server ( which we just declared above ) with the id of the application ( 4020 is Garry's Mod ) and the name of the directory we want the server to be hosted from:

update_server 4020 "server_1"

# Add any additional servers here by repeating the above, but using a different directory name.

# Exit with status code 0 ( which means OK )
exit 0

$ chmod +x ./update_gmod.sh                                             # make it executeable
$ ./update_gmod.sh

```
