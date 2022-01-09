# stratum-1-microserver-howto

This is a fork of the NTPSEC stratum-1-microserver-howto guide which has now been achived and is not beeing maintained.

I was looking for a simple way of building a GSP Disciplined Time Server based on a Raspberry Pi 4 for a Radio Telescope project.  

A GPS NTP Network Time Server is a GPS-based NTP server device that supplies accurate time for all computers and time-keeping devices.

There are a number of guides on the internet to allow you to build one but I found that the config was involved and many covered different OS and versions

The NTPSEC team last updated the project in 2018 and the Github site is now archived.

The advantage of the stratum-1-microserver-howto what the "clockmaker" python script what allowed me to build, ripdown, then rebuild a server in minutes. The scripts have been updated to support my system so milage may vary.  Most work was around supporting Python3 changes.

Raspberry Pi 4, 4Gb, 8Gb SSD
gy-gps6mv2 GPS board (based on the uBlox NEO-6M)
PPS wire connected to GPIO18

Raspberry Pi OS Lite
Release date: October 30th 2021
Kernel version: 5.10
Size: 463MB

Regards
Dave