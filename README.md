# Raspberry Pi & RPLidar 360 (A1)

This repository contains scripts and tools for using a SLAMTech RPLidar 360 (A1) with raspberry pi and ros. While some of the settings may need to change, much of this would also be relevant to the A2 and A3 Lidary units sold by SLAMTech.

![alt tag](https://github.com/avirtuos/RPiLidar/blob/master/docs/img/rplidar-360-laser-scanner.jpg?raw=true)

 > #### WARNING - RPLidar Serial Logic Voltage
 >While you may be tempted to connect the RPLidar directly to your Rasberry Pi via the serial IO headers, you should be careful doing so. The RPLidar documentation has inconsistencies in what voltage the serial logic employs. Parts of the spec say it is 3.3v (safe for Pi) and other parts indicate that the serial logic can reach peeks of 5v (no safe for Pi). I tested with a multi-meter and found fluctiations of up to 4.3v, so this repo assumes your are using a USB UART which will safely handle the 5v Serial logic. It unfortunately makes the enclosure bigger and a bit more power hungry.

## ROS Lidar Node


## ROS Hector-SLAM Node


## Enclosure

Check out my <a href='https://github.com/avirtuos/openscad/tree/master/waveshare'>RPLidary OpenSCAD repo</a> for enclosures that you can 3D print for your RPLidar and Raspberry Pi.

Like this unit that I use on my autonomous rover:

![alt tag](https://github.com/avirtuos/openscad/blob/master/lidar/docs/img/lidar_case.png?raw=true)
