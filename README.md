# ODK_Pi

A deployable ODK server based on Raspberry Pi Raspbian

**Equipment**

Raspberry Pi (Tested on R Pi 3 B+)  
SDHC Micro SD card (16 GB tested) formatted to FAT / FAT32  
Noobs https://www.raspberrypi.org/downloads/noobs/  

**R Pi Setup**  

* Install Noobs as per instructions (copy all files from noobs folder on to SDHC card and plug in to RPi)  
* Power on, connect to Wifi, install Raspbian and update

> You should have a basic rpi installation of raspbian now
  
* Open Terminal

=======
###Get current date and time from internet at startup  
```sudo nano /etc/rc.local```
  
* Add the following line to the rc.local file above *exit 0*

```date -s "$(wget -qSO- --max-redirect=0 google.com 2>&1 | grep Date: | cut -d' ' -f5-8)Z"```


###update list of packages and remove bloatware
```
sudo apt-get update  
sudo apt-get upgrade
sudo apt-get remove --purge wolfram-engine scratch2 libreoffice* scratch minecraft-pi sonic-pi dillo gpicview
```  

* select 'y' when prompted to remove bloatware

* clean up leftovers from removed packages

```
sudo apt-get clean
sudo apt-get autoremove
```

### Add required packages

```
sudo apt-get install apt-file
sudo apt-file update
```
```
sudo apt-get install libcurl4-openssl-dev libssl-dev libxml2 libxml2-dev libgdal-dev usbmount 
```
### Install JDK

```
sudo apt-get install ca-certificates-java  
sudo apt-get install openjdk-9-jre
```
###get odk briefcase

```
curl --silent "https://api.github.com/repos/opendatakit/briefcase/releases/latest" | grep "browser_download_url" | sed -E 's/.*"([^"]+)".*/\1/' |xargs wget
```

### Set Jar files to open on double click

* Add a java.desktop file to tell OS how to handle .jar files

```
nano /usr/share/applications/java.desktop 
```



```
[[Desktop Entry]
Name=Java
Comment=Java
GenericName=Java
Keywords=java
Exec=java -jar %f
Terminal=false
X-MultipleArgs=false
Type=Application
MimeType=application/x-java-archive
StartupNotify=true
```

After adding this file you should be able to find an entry called Java in the Open *file with...* -Dialog
Then open a file this way and select always open with this program

#### ODK Briefcase should now be able to download data and export with private key


###Add support for connecting android devices

``` 
sudo apt-get install gvfs-backends gvfs-bin gvfs-fuse gvfs-daemons  
```

 * Plug in and unlock an Android device
 * Find generic parent location of mount using 

```mount | grep 'gvfsd-fuse'```

* example

> gvfsd-fuse on **/run/user/1000/gvfs** type fuse.gvfsd-fuse (rw,nosuid,nodev,relatime,user_id=1000,group_id=1000)

* the key folder here is **/run/user/1000/gvfs/**
* Change to the root directory and create a symlink to the parent directory

```
cd ~
ln -s /run/user/1000/gvfs Android_Device

``` 

* Open a file manager window and select *Edit* > *Preferences* > *Volume Management*
* Unselect *"Show available options for removable media..."*

You should now have a folder called "Android_Device" in the root directory. When a device is connected to the RPI, this folder should automagically get a new subfolder for the Android MTP protocol. 

* Connect an Android device by USB cable
* Unlock the device
* Select "Yes" to any query about using USB/MTB for transfer
* If you navigate to the *Android_Device* folder you should see a folder called *mtp:host...* This has the device's file system inside
* If you don't see that folder, press F5 to refresh
* When you switch devices, press F5 to update the folder.



**ODK briefcase should now be able to pull all data from devices via the "Android_Devices" Folder.**












##APPENDIX

### If you want to run a shell script by double clicking

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


Push forms to pi through sftp on "andFTP" app. hostname = ip of device username pi  password FMTC	 
don't change anything else.



###Install R and pandoc
Useful for analysis (R) and reporting (Pandoc)
```sudo apt-get install r-base r-base-dev pandoc pandoc-citeproc```


### change priority of WIFI networks
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

###For UK academics on Eduroam
Eduroam is a UK wide wifi provider for cross-institutional working. It can be hard to set up on Rpi, but instructions [here](http://jankuester.com/connecting-raspberry-pi-to-eduroam-using-wpa-supplicant/) should be helpful. Below worked for me.

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

###Hotswitching between networks
Linux provides a nice easy way to switch between networks. A simple sh script would provide a nice clickable way to switch via a GUI, but on RPi you get the added bonus that you can connect physical switches to the pins. A push button could be used to switch between wifi networks (i.e. to move from a fully online network to an offline local wifi network provided by a hotspot. The latter is very useful if you want to use VNC software to allow using an Android tablet as a screen and interface to RPi whilst in the field. 


* To shift from one network to another 

```wpa_cli select_network 0``` or number of network is all you need to do. 

###Set up github
Github provides a nice way to get updated analysis scripts on to the RPi automatically.

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













System setup 

Need the following on tablet
VNC Viewer app
Net Analyser


Have to enable SSH and VNC server on RPI through sudo raspi-config
Also install network analyser to find out what ip is of rpi


  

  
  
  
 git status
  
###Dropbox 
Dropbox is a convenient way to get data back off an Rpi, though not recommended for sensitive data (at the very least you should encrypt the data).

Dropbox integration for RPi is not great, but [Dropbox Uploader](https://github.com/andreafabrizi/Dropbox-Uploader) is an excellent script for moving stuff to and from Dropbox. [These instructions](https://www.raspberrypi.org/magpi/dropbox-raspberry-pi/) are also very useful.

* Visit [Dropbox Uploader](https://github.com/andreafabrizi/Dropbox-Uploader) follow the instructions in the README.MD for the most up to date instructions.   

  


To clone RPI SD Card
`sudo dd if=/dev/disk3 of=/Volumes/Totoro/Users/chrissyhroberts/Dropbox/RPI_Backups/20180704/A20180704 bs=1m`

To restore RPI SD Card
`sudo dd if=/Volumes/Totoro/Users/chrissyhroberts/Dropbox/RPI_Backups/20180704/A20180704 of=/dev/disk3 bs=1m`

based on this example 

backup:

dd if=/dev/sdb of=sd.img bs=4M

restore:

dd if=sd.img of=/dev/sdb bs=4M


#github desktop for linux
https://github.com/shiftkey/desktop/releases

How to automatically mount an USB device on plugin-time on an already running system
sudo apt-get install usbmount  







