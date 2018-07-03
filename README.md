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
sudo apt-get update
sudo apt-get install r-base r-base-dev 
R
install.packages("tidyverse")
sudo apt-get install openjdk-11-jdk
```



