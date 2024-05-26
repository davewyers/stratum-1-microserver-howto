# Stratum-1-Microserver HOWTO

This is a fork of the **NTPSEC stratum-1-microserver-howto** guide by Eric S. Raymond which has now been achived and does not appear to be maintained.

I was looking for a simple way of building a GSP Disciplined Time Server based on a Raspberry Pi 4 for a Radio Telescope project.  

A GPS NTP Network Time Server is a GPS-based NTP server device that supplies accurate time for all computers and time-keeping devices.

There are a number of guides on the internet to allow you to build one but I found that the config was involved and many covered different OS and software versions

The NTPSEC team last updated the Howto Github repository in 2017 and the repository is now archived.

From my perspective the advantage of the stratum-1-microserver-howto was the "clockmaker" python script what allowed me to build, ripdown, then rebuild a server in minutes. The scripts have been updated to support my system so milage may vary.  

Most of the work was around supporting Python3 and PI OS changes since 2017.

## Hardware
I have validated these steps with the following hardware, 
 - Raspberry PI 4, 
    - Quad core ARM Cortex-A72 64-bit SoC @ 1.5GHz and 4Gb of SDRAM
 - Raspberry PI Zero W 2, 
     - Quad-core 64-bit ARM Cortex-A53 64-bit SoC @ 1.0GHz and 512MB of SDRAM
 
In both cases I was using the same GPS module 
 - **gy-gps6mv2** GPS board (based on the uBlox NEO-6M)
 - PPS wire connected to GPIO18
    - See the **gy-gps-pps-fix.jpg** file to see where to connect the wire

## Software

 - Client PC
     -  Windows 10 Pro
     -  Raspberry Pi Imager v1.8.5
     -  Putty SSL Client 0.74
 - Raspberry PI
     - Raspberry Pi OS Lite (54-bit)
     - Release date: 2024-03-15

*****
# Stratum-1-Microserver
## Guide for Windows users

The following are rough notes to get you through the set up from a windows workstation.

## Create Disk Image
Download the latest version of the Raspberry PI Imager software from the Raspberry PI web site

> https://downloads.raspberrypi.org/imager/imager_latest.exe

I have used the Lite version (no GUI) but you can use the full version if you have the 4Gb or 8Gb boards.

Running the Raspberry Lite OS with the Time Service uses approx 50Mb of RAM including the running OS, as such Time Service runs easily on the PI Zero W 2 with 512Mb

Use The PI Imager to create new boot image for the SD card.  
Select
 - Device = Raspberry Pi Zero 2 W
 - OS = Raspberry Pi OS Lite (64-bit)
 - Storage = your SD Card

When asked for OS Customisation Options, select Yes
Under General
 - Set Hostname to "PI-Time",
 - Set Username & Password
 - Set Wifi SSID & Password
 - Set Locale
Under Services
 - tick Enable SSL and use password authentication.

Click Save
Select Yes


Once finished insert the SD card into the Raspberry PI and boot up the PI.

On the Windows client open PuTTY and SSH into the server "PI-Time", using username "pi" & the password that you set above.


## config static IP address (Optional)
If you would like to set a static IP you can do so as below, this is not required as Putty will open a SSL connection using the hostname "PI-Time"

Old Method

Open the config file.

> sudo nano /etc/dhcpcd.conf

Add following lines to the end of the file.

> interface eth0
> 
> static ip_address=192.168.33.25/24
> 
> static routers=192.168.33.1
> 
> static domain_name_servers=192.168.33.1

Save and exit from Nano by using Ctrl-X, "Y" to save, then Enter to save. 


New Method
https://www.abelectronics.co.uk/kb/article/31/set-a-static-ip-address-on-raspberry-pi-os-bookworm


*****

# Clockmaker
Clockmaker is the Python script that automates 90% of the effort here.

A full understanding of the original script can be found here,

> https://www.ntpsec.org/white-papers/stratum-1-microserver-howto/

**The updated forked version that I am working from can be found at** 

> https://github.com/davewyers/stratum-1-microserver-howto/blob/master/clockmaker

Log into the PI server via SSL, then run the following commands to download the current version of the script and set the permissions to allow execution.

> cd \~
> 
> mkdir clockmaker
> 
> cd clockmaker
> 
> wget https://raw.githubusercontent.com/davewyers/stratum-1-microserver-howto/master/clockmaker
> 
> sudo chmod a+x clockmaker

It is now time to run the **clockmaker** config process to set up the PI system to support a serial GPS board.

> sudo ./clockmaker --config

Select the Uptronics GPS option to use GPIO pin 18 for the PPS input

When complete the system will reboot.

Once you have logged back in to the PI, run the **clockmaker** build process to download the setup files to your local machine.

> cd clockmaker
> 
>   ./clockmaker --build

If required edit the ntp.conf template file based on your location

> sudo nano ./ntp.conf

Edit the following lines to pick the correct upstream POOL location, in my case I have set the NZ pool servers but you will use the time servers for your location.

> \# NZ Servers
> 
> pool nz.pool.ntp.org iburst maxpoll 5

Then save and exit nano.

Now it is time to run the install process to copy these files to the correct locations allowing the Time Service to run on start up of the server

> sudo ./clockmaker --install

**optional**
**WARNING before running this you must have another account to log in as the default PI account will be removed**

> sudo ./clockmaker --mask
> 
> sudo clockmaker --secure
> 
> sudo clockmaker --strip

Once the system is up and running you can ensure that the system stays upto date with the current software by running  

> sudo clockmaker --update

*****

**Monitoring clients**
There are a number of applications that can be run to monitor the system.  These do not need root / sudo access to run.

You may need to add the following to your ~/.bashrc file

> export PYTHONPATH=${PYTHONPATH}:/usr/local/lib/python3/dist-packages/

 - ntpq -p -u
 - ntpmon -u
 - cgps -u m
 - gpsmon
 - ~/clockmaker/ntpoffset

*****
**Manually start time keeping**
If you need to check or restart the services, firstly kill the existing processes then restart, a reboot should also work.

> sudo killall -9 gpsd ntpd
> 
> sudo gpsd -n /dev/ttyAMA0
> 
> sudo ntpd -gN



**Other Resources**
 - https://gpsd.gitlab.io/gpsd/gpsd-time-service-howto.html#_performance_tuning
 - http://www.gregledet.net/computers/building-a-stratum-1-ntp-server-with-a-raspberry-pi-4-and-adafruit-ultimate-gps-hat/
 - https://weberblog.net/setting-up-nts-secured-ntp-with-ntpsec/
 - https://psychogun.github.io/docs/linux/Stratum-1-NTP-Server-using-Raspberry-Pi/
 - https://www.satsignal.eu/ntp/Raspberry-Pi-quickstart.html
 - https://www.techsolvency.com/ntp/

