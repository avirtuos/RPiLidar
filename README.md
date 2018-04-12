# Raspberry Pi & RPLidar 360 (A1)

This repository contains scripts and tools for using a SLAMTech RPLidar 360 (A1) with raspberry pi and ros. While some of the settings may need to change, much of this would also be relevant to the A2 and A3 Lidary units sold by SLAMTech.

![alt tag](https://github.com/avirtuos/RPiLidar/blob/master/docs/img/rplidar-360-laser-scanner.jpg?raw=true)

 > #### WARNING - RPLidar Serial Logic Voltage
 >While you may be tempted to connect the RPLidar directly to your Rasberry Pi via the serial IO headers, you should be careful doing so. The RPLidar documentation has inconsistencies in what voltage the serial logic employs. Parts of the spec say it is 3.3v (safe for Pi) and other parts indicate that the serial logic can reach peeks of 5v (no safe for Pi). I tested with a multi-meter and found fluctiations of up to 4.3v, so this repo assumes your are using a USB UART which will safely handle the 5v Serial logic. It unfortunately makes the enclosure bigger and a bit more power hungry.

## Testing RPLidar

##### 1. Setup /dev/ttyUSBX


Check which tty your serial RPLidar 

```bash
sudo gpasswd --add ${USER} dialout
```





## ROS Lidar Node

Follow the <a href='http://wiki.ros.org/ROSberryPi/Installing%20ROS%20Kinetic%20on%20the%20Raspberry%20Pi'>instructions here</a> to install ros kenetic 'comm' only on the raspberry pi. I don't recommend the full or desktop install, it will add a lot of dependencies you don't need or want for this setup.

##### 1. You'll need set up a few dependencies and add the ARM ROS Repos


```bash
mkdir ros_install
cd ros_install
sudo apt-get update
sudo apt-get install dirmngr
sudo apt-get install libxml2-dev libxml2
wget http://sourceforge.net/projects/collada-dom/files/latest/download
tar -xvf download
cd collada-dom-2.4.0

#You may need to manually apply the changes in this commit, its a one line change. 
#https://github.com/bmwcarit/meta-ros/commit/dbc17a263d4d948cd8069e83bfa7b0a614ab7555
cmake . && sudo make install

sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
sudo apt-key adv --keyserver hkp://ha.pool.sks-keyservers.net:80 --recv-key 421C365BD9FF1F717815A3895523BAEEB01FA116
sudo apt-get update
```

This dependency is optional, i skipped it for the pi zero as it required several patches to get compiling.

```bash
mkdir -p ~/ros_catkin_ws/external_src
cd ~/ros_catkin_ws/external_src
wget http://sourceforge.net/projects/assimp/files/assimp-3.1/assimp-3.1.1_no_test_models.zip/download -O assimp-3.1.1_no_test_models.zip
unzip assimp-3.1.1_no_test_models.zip
cd assimp-3.1.1
cmake .
make
sudo make install
```

Follow the installation instructions <a href='http://wiki.ros.org/kinetic/Installation/Source'>here</a>, and use the below command to preparing which ROS componets to install/compile:

```bash
rosinstall_generator robot --rosdistro kinetic --deps --wet-only --exclude collada_parser collada_urdf --tar > kinetic-robot-wet.rosinstall

wstool init -j1 -l1 src kinetic-robot-wet.rosinstall

rosdep install -y --from-paths src --ignore-src --rosdistro kinetic -r --os=debian:jessie
```

Next we need to install the required components for hector_slam. Do this in a fresh catkin workspace. The question and associated answer in <a rhef='https://answers.ros.org/question/205853/how-to-add-cv_bridge-and-image_transport-package-to-compile-in-raspberry-pi/'>this post</a> were a good guide.

```bash
cd ~
mkdir ros_slam
mkdir ros_slam/src
cd ros_slam
catkin_make 
rosinstall_generator image_transport laser_geometry cv_bridge --rosdistro kinetic --deps --wet-only --tar > kinetic-robot-wet.rosinstall
wstool init src kinetic-robot-wet.rosinstall
rosdep install -y --from-paths src --ignore-src --rosdistro kinetic -r --os=debian:jessie
sudo ./src/catkin/bin/catkin_make_isolated -j1 -l1 --install -DCMAKE_BUILD_TYPE=Release --install-space /opt/ros/kinetic
```

Now lets install hector_mapping, hector_trajectory_server, hector_geotiff
```bash
cd ~
mkdir ros_dep_install
mkdir ros_dep_install/src
cd ros_dep_install
catkin_make 
rosinstall_generator hector_mapping hector_trajectory_server hector_geotiff --rosdistro kinetic --deps --wet-only --tar > kinetic-dep-wet.rosinstall
wstool init src kinetic-dep-wet.rosinstall
rosdep install -y --from-paths src --ignore-src --rosdistro kinetic -r --os=debian:jessie
sudo ./src/catkin/bin/catkin_make_isolated -j1 -l1 --install -DCMAKE_BUILD_TYPE=Release --install-space /opt/ros/kinetic
```


##### 2. Setup a new catkin workspace and build the rplidar_ros package

```bash
mkdir /home/pi/workspace
cd /home/pi/workspace
mkdir src
catkin_make
source devel/setup.bash
cd src
wget https://github.com/robopeak/rplidar_ros/archive/master.zip
unzip master.zip
rm master.zip
mv rplidar_ros-master rplidar_ros
cd ..

#Needed to properly install sensor_msgs (seems like a raspberry pi jessie specific issue)
rosinstall_generator sensor_msgs --rosdistro kinetic --deps --wet-only --tar > 
kinetic-sensor_msgs-wet.rosinstall

wstool init src kinetic-sensor_msgs-wet.rosinstall

catkin_make
```

##### 3. Environment Variables

On the host running roscore, you'll want to make sure the following items are set in the env before sourcing the ros env files.

```bash
if [ -f ~/.bashrc ]; then
        source ~/.bashrc
fi

export PRIMARY_IP_ADDR=`ifconfig | sed -En 's/127.0.0.1//;s/.*inet (addr:)?(([0-9]*\.){3}[0-9]*).*/\2/p'`
export ROS_IP=$PRIMARY_IP_ADDR
```

On the host running the rplidar node, you'll want to add the following:

```bash
export ROS_MASTER_URI=http://pi-ros-core.localdomain:11311
```


## ROS Hector-SLAM Node
asd

## Enclosure

Check out my <a href='https://github.com/avirtuos/openscad/tree/master/waveshare'>RPLidary OpenSCAD repo</a> for enclosures that you can 3D print for your RPLidar and Raspberry Pi.

Like this unit that I use on my autonomous rover:

![alt tag](https://github.com/avirtuos/openscad/blob/master/lidar/docs/img/lidar_case.png?raw=true)
