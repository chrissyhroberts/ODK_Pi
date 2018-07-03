# ODK_Pi

A deployable ODK server based on Raspberry Pi, R, Raspbian

**Equipment**

Raspberry Pi (Tested on R Pi 3 B+)
SDHC Micro SD card (16 GB tested) formatted to FAT / FAT32
Noobs https://www.raspberrypi.org/downloads/noobs/

**R Pi Setup**
Install Noobs as per instructions (copy all files from noobs folder on to SDHC card and plug in to RPi)
Connect to Wifi and update


Open terminal

```
#add current date at startup
	sudo nano /etc/rc.local

#add the following line to the rc.local file

sudo date -s "$(wget -qSO- --max-redirect=0 google.com 2>&1 | grep Date: | cut -d' ' -f5-8)Z"

#update list of packages
sudo apt-get update  

#remove bloating packages in raspbian
sudo apt-get remove --purge wolfram-engine scratch nuscratch sonic-pi pistore idle3 smartsim penguinspuzzle java-common minecraft-pi python-minecraftpi python3-minecraftpi libx11-6 libgtk-3-common xkb-data lxde-icon-theme raspberrypi-artwork 

#install R 
sudo apt-get install r-base r-base-dev 

#start R environment and install packages
R
install.packages("tidyverse")
q()

#install java development kit
sudo apt-get install default-jdk
```

**Run a Script after login**  
How to automatically run a script after login.  
	
	#Step 1: Open a terminal session and edit the file /etc/profile
	sudo nano /etc/profile
	
	#Step 2: Add the following line to the end of the file  
	./home/pi/your_script_name.sh

	#Step 3: Save and Exit
	#Press Ctrl+X to exit nano editor followed by Y to save the file.


