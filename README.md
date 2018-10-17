# ODK_Pi

A deployable ODK Briefcase based data hub based on Raspberry Pi Raspbian

#### Background
Some of our users are working in areas with little or no internet connection, so a portable hub for aggregating data in the field, which is able to later push data to ODK aggregate is a useful interim/backup device that can also be preconfigured to output CSVs, PDF copies of data and so on for on the ground data needs. 

The ODK briefcase software provides all the functionality we need for this purpose. The benefits of putting briefcase on to a Raspberry Pi (RPi) are 

* The small size and weight of the RPi mean that you could send it in the post, carry it in your pocket and include it as a 'just in case' plan B measure for studies where your baggage allowance is only what you can carry.
* Dedicated system means you don't have to 'donate' your own laptop to study team for the duration of the work.
* The RPi costs ~£35 and can be used with a tablet/phone/TV as display and touchscreen/keyboard/mouse as I/O devices
* Runs off pretty much any power source and has low consumption and heat output. Can be run inside a tupperware box, so good off grid, in the rainforest and so on.
* Provides full linux system including encryption/decryption, R, Python, Java etc. 
* SD cards can be cloned, meaning that you can take a box of RPis and set up as many hubs as you need if the scale of project grows.

These instructions should be sufficient to set up a new installation of the Raspbian operating system, then to install any necessary packages and software (including ODK Briefcase). It will also help you to set up the system so that you can work with Android devices via the MTP protocol (which is not especially obvious) and various other tips and tricks that make all these things fit together.

Nothing in this guide is particularly difficult and not novel, but I think this is the first place where everything is listed as a single set of instructions. 


**Equipment**

[Raspberry Pi](https://www.raspberrypi.org/) (Tested on R Pi 3 B+)  
SDHC Micro SD card (16 GB tested) formatted to FAT / FAT32  
Noobs [https://www.raspberrypi.org/downloads/noobs/](https://www.raspberrypi.org/downloads/noobs/)  

**R Pi Setup**  

* Install Noobs as per [instructions](https://www.raspberrypi.org/downloads/noobs/). Tested on Noobs Lite (Network install)

	1) Copy all files from noobs folder on to SDHC card  
	2) Plug SD card in to RPi. 
	

* Power on the RPi, connect to Wifi, follow steps to install Raspbian and update

**You should now have a basic RPi installation of raspbian**

* Fiddle around with your wifi settings, bluetooth etc.  
* Open Terminal

=======
### Get current date and time from internet at startup  
The RPi doesn't have a real time clock onboard, so I'd recommend putting on a clock board if using this in the field. Otherwise you'll be relying on internet time. For the time being though, let's stick with the basic board and grab internet time at startup. 

```sudo nano /etc/rc.local```
  
* Add the following line to the rc.local file above where it says *exit 0*

```date -s "$(wget -qSO- --max-redirect=0 google.com 2>&1 | grep Date: | cut -d' ' -f5-8)Z"```


### update list of packages and remove bloatware
I'm guessing you don't want to play Minecraft on this RPi (though it is a great game) so let's save about 1.5 GB of SD card space and remove bloatware that comes preinstalled on this version of Raspbian.

```
sudo apt-get update  
sudo apt-get upgrade
sudo apt-get remove --purge wolfram-engine scratch2 libreoffice* scratch minecraft-pi sonic-pi dillo gpicview
```  

* select 'y' or 'Y' when prompted to remove these packages
* clean up leftover dependencies from removed packages

```
sudo apt-get clean
sudo apt-get autoremove
```

### Add required packages
This will add a bunch of useful things that you need

```
sudo apt-get install apt-file
sudo apt-file update
```

```
sudo apt-get install libcurl4-openssl-dev libssl-dev libxml2 libxml2-dev libgdal-dev usbmount 
```
### Install Java Development Kit (JDK)

```
sudo apt-get install ca-certificates-java  
sudo apt-get install openjdk-9-jre
```

### Get odk briefcase

```
curl --silent "https://api.github.com/repos/opendatakit/briefcase/releases/latest" | grep "browser_download_url" | sed -E 's/.*"([^"]+)".*/\1/' |xargs wget
```

### Set Jar files to open on double click
You probably don't want to have to start ODK Briefcase from the command line every time you use it, so this will make jar files run by double-clicking them

* Add a java.desktop file to tell OS how to handle .jar files

```
sudo nano /usr/share/applications/java.desktop 
```

```
[[Desktop Entry]
Name=Java
Comment=Java
GenericName=Java
Keywords=java
Exec=/usr/bin/java -jar %f
Terminal=false
X-MultipleArgs=false
Type=Application
MimeType=application/x-java-archive
StartupNotify=true
```
Close the *'nano'* text editor by pressing *CTRL* & *C* and then press *"Y"* to save the new file

After adding this file you should be able to find an entry called Java in the *Open file with...* dialog when you right click the jar file.

Select *always open with this program* and you are set to go.


ODK Briefcase should now be able to download data and export decrypted data when provided with a private key (.pem).


### Add support for connecting android devices
Android uses the MTP protocol to connect to linux. This is no doubt safer but is a lot *weirder* than the old fashioned approach where it just mounted your tablet as a USB drive on your computer. Without a bit of jiggery pokery it's hard to figure out how to find the folders. The next few steps make it much easier. 

``` 
sudo apt-get install gvfs-backends gvfs-bin gvfs-fuse gvfs-daemons  
```

 * Plug in and unlock an Android device
 * Find generic parent location of mount using 

```mount | grep 'gvfsd-fuse'```

* example

> gvfsd-fuse on **/run/user/1000/gvfs** type fuse.gvfsd-fuse (rw,nosuid,nodev,relatime,user_id=1000,group_id=1000)

* the key folder here is **/run/user/1000/gvfs/** but it might be different on your system
* Change to the root directory and create a symlink to the parent directory

```
cd ~
ln -s /run/user/1000/gvfs Android_Device

``` 

* Open a file manager window and select *Edit* > *Preferences* > *Volume Management*
* Unselect *"Show available options for removable media..."*

You should now have a folder called "Android_Device" in the root directory. When a device is connected to the RPi, this folder should automagically get a new subfolder for the Android MTP protocol. 

* Connect an Android device by USB cable
* Unlock the device
* Select "Yes" to any query about using USB/MTB for transfer
* If you navigate to the *Android_Device* folder you should see a folder called *mtp:host...* This has the device's file system inside
* If you don't see that folder, press F5 to refresh
* When you switch devices, press F5 to update the folder.


**ODK briefcase should now be able to pull all data from devices via the "Android_Devices" Folder.** 

Navigate to it and you'll see a uniquely named folder under which you should see the internal and SD card folders of your device. Inside one or the other will be your *ODK* folder. That's the one that ODK Briefcase can pull instances from.

### Wireless push of instances from Android to RPi

You can send forms to the RPi through sftp using the excellent **[andFTP](https://play.google.com/store/apps/details?id=lysesoft.andftp&hl=en_US)** 


### If you want to run a shell script by double clicking
This can be useful if you have downstream stuff going on, or if you want to write a script to control a batch of ODK briefcase operations from the command line interface (CLI). 

* Create a *.desktop file in your /usr/share/applications
* Example - *do.stuff.sh*

```nano /usr/share/applications/do.stuff.desktop```

```
[Desktop Entry]
Version=1.0  
Exec=/home/yourname/bin/do.stuff.sh  
Name=SSH Server  
GenericName=SSH Server  
Comment=Run the do stuff script  
Encoding=UTF-8  
Terminal=true  
Type=Application  
Categories=Application;Network;
```

Use *open with...* and *set as default* and then script will run on double click


### Install R and pandoc
Useful for analysis (R) and reporting (Pandoc)
```sudo apt-get install r-base r-base-dev pandoc pandoc-citeproc```


### Change priority of WIFI networks
This can be useful if you want to control the precedence of networks joined over wifi. 

```sudo nano /etc/wpa_supplicant/wpa_supplicant.conf```

Add priority=2 to the *wifi_B* block and priority=1 to the *wifi_A* block in the  file.
Higher number is higher priority so here *wifi_B* will be joined by preference
  

```
network={  
    ssid = "wifi_A"  
    psk = "passwordOfA"  
    priority = 1  
}  
network={  
   ssid = "wifi_B"  
   psk = "passwordOfB"  
   priority = 2  
}
```

### For UK academics on Eduroam
Eduroam is a UK wide wifi provider for cross-institutional working. It can be hard to set up on RPi, but instructions [here](http://jankuester.com/connecting-raspberry-pi-to-eduroam-using-wpa-supplicant/) should be helpful. Below worked for me.

```sudo nano /etc/wpa_supplicant/wpa_supplicant.conf```
 
```
ctrl_interface=/var/run/wpa_supplicant
ctrl_interface_group=netdev

country=GB

network={
ssid="eduroam"
key_mgmt=WPA-EAP
eap=PEAP
phase2="auth=MSCHAPV2"
identity="username@youruniversity.ac.uk"
anonymous_identity="anonymous@youruniversity.ac.uk"
password="pwd"
}
```

### Hotswitching between networks
Linux provides a nice easy way to switch between networks. A simple sh script would provide a nice clickable way to switch via a GUI, but on RPi you get the added bonus that you can connect physical switches to the pins. A push button could be used to switch between wifi networks (i.e. to move from a fully online network to an offline local wifi network provided by a hotspot. The latter is very useful if you want to use VNC software to allow using an Android tablet as a screen and interface to RPi whilst in the field. 


* To shift from one network to another 

```wpa_cli select_network 0``` or number of network is all you need to do. 

### Set up github
Github provides a nice way to get updated analysis scripts on to the RPi automatically.
Git is built in to the Raspbian OS, so you just need to connect to GitHub to get your scripts

```
git config --global user.name "USERNAME"  
git config --global user.email "you@youremail.com"  
git config --global core.editor nano
```

You could add a git pull script at startup to ensure that all scripts are updated on startup. Alternatively you could set up a cron task. 

To run a script after login    

```
sudo nano /etc/profile
```
	
Add the following line to the end of the file  
 
```
./home/pi/your_script_name.sh
```
  
### Dropbox 
Dropbox is a convenient way to get data back off an RPi, though not recommended for sensitive data (at the very least you should encrypt the data).

Dropbox integration for RPi is not great, but [Dropbox Uploader](https://github.com/andreafabrizi/Dropbox-Uploader) is an excellent script for moving stuff to and from Dropbox. [These instructions](https://www.raspberrypi.org/magpi/dropbox-raspberry-pi/) are also very useful.

* Visit [Dropbox Uploader](https://github.com/andreafabrizi/Dropbox-Uploader) follow the instructions in the README.MD for the most up to date instructions.   


### Backup your SD card so that you can use it elsewhere or recover from mishaps

**BEWARE** that you have to use the right mount point (in the example I used /dev/disk3). If you try to write to the wrong disk when restoring the SD card, then you will probably brick your system.

Run ```df``` to see what mounts you have. Identify which one is the disk you want to restore to. 

To clone RPi SD Card
`sudo dd if=/dev/disk3 of=/Volumes/RPi_Backups/name bs=1m`

To restore RPi SD Card
`sudo dd if=/Volumes/RPi_Backups/name of=/dev/disk3 bs=1m`

### Add a PiTFT screen

Tested using the AdaFruit Assembled 480x320 3.5" TFT+ Touchscreen (Resistive)

* To install the screen
	* Shut down the RPi
	* Plug the screen on to the GPIO pins of the RPi
	* Boot up to Raspbian 

```
cd ~
wget https://raw.githubusercontent.com/adafruit/Raspberry-Pi-Installer-Scripts/master/adafruit-pitft.sh
chmod +x adafruit-pitft.sh
sudo ./adafruit-pitft.sh
```
The script will attempt to install the screen automatically. When prompted to select the configuration, choose the screen you are installing, for instance
> 4. PiTFT 3.5" resistive touch (320x480)

Next you need to select the screen rotation you want, for instance
> 1. 90 Degrees (Landscape)

The next two steps determine the best setting for the screen

>Would you like the console to appear on the PiTFT display?

This option is good if you want to use the screen as a console window. There's no HDMI output with this config and to be honest I think it is not what most people want, so select ```No``` to this one.  

>Would you like the HDMI display to mirror to the PiTFT display?  

This option will mirror the HDMI output to the PiTFT screen, so is the better choice for most people. Select ```Yes``` to this option.

After rebooting, the screen should start working, with some fairly clunky but functional touch support. (Hint : Use a pointy bit of plastic to get screen to work as a mouse)

**Set up screen to work better**

* Open terminal then go to ```Edit``` menu, followed by ```Preferences```.
* Change the font to something a bit clearer like ```Monospace 14 Bold```


Add some commands to the ```bin``` folder to allow hotswitching of the screen (saves power if you don't need it on).

* Make a command for turning screen off

```
sudo nano /usr/bin/scroff
```

Add this line to the file

```
sudo sh -c 'echo "0" > /sys/class/backlight/soc\:backlig
ht/brightness'
```
press ```CTRL + C``` and ```Y``` to save the file and close nano

* Make a command for turning screen on

```
sudo nano /usr/bin/scron
```

Add this line to the file

```
sudo sh -c 'echo "1" > /sys/class/backlight/soc\:backlig
ht/brightness'
```
press ```CTRL + C``` and ```Y``` to save the file and close nano



Make both commands executable
```
sudo chmod +x /usr/bin/scroff
sudo chmod +x /usr/bin/scron
```

Then test that they work  
```
scroff
```
should turn the screen off and ```scron``` should turn it on again.
 
* Add a software keyboard  
Quite hard to use on tiny screen, but could be useful in a pinch

```
sudo apt-get install matchbox-keyboard
sudo apt-get install libmatchbox1 -y 
sudo nano /usr/bin/toggle-matchbox-keyboard.sh
```

Add this text to the ```toggle-matchbox-keyboard.sh``` file. 

```
#!/bin/bash
#This script toggle the virtual keyboard

PID=`pidof matchbox-keyboard`
if [ ! -e $PID ]; then
  killall matchbox-keyboard
else
 matchbox-keyboard&
fi

``` 
Then close file and make executable

```
sudo chmod +x /usr/bin/toggle-matchbox-keyboard.sh

```

Keyboard can then be accessed through ```menu``` > ```Accessories``` > ```Keyboard```

``````	 
