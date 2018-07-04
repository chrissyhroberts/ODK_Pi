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


<<<<<<< HEAD
#update list of packages
sudo apt-get update  

#install R 
sudo apt-get install r-base r-base-dev 

#start R environment and install packages
R
install.packages("tidyverse")
q()

#install java development kit
`sudo apt-get install default-jdk`



change priority of WIFI networks
Add priority=2 to the wifi_B block and priority=1 to the wifi_A block in the /etc/wpa_supplicant/wpa_supplicant.conf file.
Higher number is higher priority

i.e.
`network={
    ssid = "wifi_A"
    psk = "passwordOfA"
    priority = 1
}
network={
   ssid = "wifi_B"
   psk = "passwordOfB"
   priority = 2
}
`

to shift from one network to another 

`wpa_cli select_network 0`



System setup 

Need the following on tablet
VNC Viewer
Net Analyser



**Run a Script after login**  
=======
#add current date at startup  
`	sudo nano /etc/rc.local`    
  
#add the following line to the rc.local file  

`sudo date -s "$(wget -qSO- --max-redirect=0 google.com 2>&1 | grep Date: | cut -d' ' -f5-8)Z"`  
  
#update list of packages  
`sudo apt-get update  `  

#install java development kit  
sudo apt-get update && sudo apt-get install oracle-java7-jdk  
cd /usr/lib/jvm/jdk-8-oracle-arm32-vfp-hflt/jre/lib/security  
http://www.oracle.com/technetwork/java/javase/downloads/jce8-download-2133166.html  

this bit gets odk briefcase working  
sudo apt-get remove openjdk-8-jre-headless openjdk-8-jre  
sudo apt-get install ca-certificates-java  
sudo apt-get install openjdk-8-jre-headless  
sudo apt-get install openjdk-8-jre # Optional, enables Java GUI apps  



#install R   
`sudo apt-get install r-base r-base-dev `  
pandoc  
sudo apt-get install pandoc pandoc-citeproc  
  
#start R environment and install packages  
`R  
install.packages("tidyverse")  
q()  
`  
  
**Run a Script after login**    
>>>>>>> b5143f658c282c2c197c8f0ea83da59e7cefb219
How to automatically run a script after login.  
	
	#Step 1: Open a terminal session and edit the file /etc/profile
	sudo nano /etc/profile
	
	#Step 2: Add the following line to the end of the file  
	./home/pi/your_script_name.sh

	#Step 3: Save and Exit
	#Press Ctrl+X to exit nano editor followed by Y to save the file.
s


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
