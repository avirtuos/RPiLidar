# Raspberry Pi & RPLidar 360 (A1)

This repository contains scripts and tools for using a SLAMTech RPLidar 360 (A1) with raspberry pi and ros. While some of the settings may need to change, much of this would also be relevant to the A2 and A3 Lidary units sold by SLAMTech.

![alt tag](https://github.com/avirtuos/RPiLidar/blob/master/docs/img/rplidar-360-laser-scanner.jpg?raw=true)


## Software Installation (ROS + RPLidar SDK)

##### 1. Setup /dev/ttyUSBX

Check which tty your serial RPLidar is attached to ensure you don't need any special kernel modules for the ftdi  or serial adapter you are using then add the your current user to have access to that TTY with the below command. You may need to log out and back in for it to take affect. You can confirm by running id ${USER} and seeing if the dialout group appears in the list after running the command.

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
mkdir ros_dep_install
mkdir ros_dep_install/src
cd ros_dep_install
catkin_make 
rosinstall_generator image_transport laser_geometry cv_bridge hector_geotiff_plugins --rosdistro kinetic --deps --wet-only --tar > kinetic-robot-wet.rosinstall
wstool init src kinetic-robot-wet.rosinstall
rosdep install -y --from-paths src --ignore-src --rosdistro kinetic -r --os=debian:jessie
sudo ./src/catkin/bin/catkin_make_isolated -j1 -l1 --install -DCMAKE_BUILD_TYPE=Release --install-space /opt/ros/kinetic
```


Now lets install hector_mapping, hector_trajectory_server, hector_geotiff, and hector_geotiff_plugins

```bash
cd ~
mkdir ros_dep_install
mkdir ros_dep_install/src
cd ros_dep_install
catkin_make 
rosinstall_generator hector_mapping hector_trajectory_server hector_geotiff hector_geotiff_plugins --rosdistro kinetic --deps --wet-only --tar > kinetic-dep-wet.rosinstall
wstool init src kinetic-dep-wet.rosinstall
rosdep install -y --from-paths src --ignore-src --rosdistro kinetic -r --os=debian:jessie
sudo ./src/catkin/bin/catkin_make_isolated -j1 -l1 --install -DCMAKE_BUILD_TYPE=Release --install-space /opt/ros/kinetic
```

##### 2. Check out this package and build it

```bash
mkdir ~/maps
mkdir ~/workspace
cd ~/workspace
wget https://github.com/avirtuos/RPiLidar/archive/master.zip
unzip master.zip
rm master.zip
mv RPiLidar-master/ ./
rm -rf RPiLidar-master
source devel/setup.bash
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

##### 4. Run the LIDAR + SLAM Nodes

After your start roscore on your intended core node, you can start up the rplidar + slam nodes.

```bash
cd ~/workspace
source devel/setup.bash
roslaunch virtuoso-rpilidar run_slam.launch
```


## Enclosure

Check out my <a href='https://github.com/avirtuos/openscad/tree/master/waveshare'>RPLidary OpenSCAD repo</a> for enclosures that you can 3D print for your RPLidar and Raspberry Pi.

Like this unit that I use on my autonomous rover:

![alt tag](https://github.com/avirtuos/openscad/blob/master/lidar/docs/img/lidar_case.png?raw=true)
