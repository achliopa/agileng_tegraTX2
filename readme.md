# NVIDIA Jetson Tegra Tx2 Setup Steps

## Bring Up the Board

* Prereqs (Jetpack 4.3)
	* Host Running Ubuntu 18.04 (Not On VM) w/ 8GB RAM and >1400x900 resolution, >30GB of HDD space
	* NVIDIA SDK Manager dowloaded on Host (from developer.nvidia.com)
	* A developer account in NVIDIA
	* A (reliable) USB cable connection between Host and microUSB port on Tegra
* Device must be in USB Recovery Mode
	* Remove Power
	* Reconnect Power
	* Press POWERBTN
	* Press and Hold REC
	* Press and Release RST
	* Keep Holding REC for 2more sec. then Release
* To confirm the Device is in Recovery Mode we run `lsusb` on host. we should see an NVIDIA device on the list
* go to the dir where sdkmanager is downloaded and run `sudo apt install ./sdkmanager-<version>`
* run sdk manager `sdkmanager`
* follow steps. download,install,flash
* when Jetson is Flashed and boots L4T enter a user account and check its IP connection to the LAN `ifconfig`
* Enter the IP and username:password to SDKManager to install Libraries on Jetson (prefer Wired LAN connection over WiFi to save time)

## Install ROS (Robot Operating System) on Tegra

* We will install Melodic Morenia as it matches our Ubuntu 18.04 LTS  distribution on Device
* we consult our [ROS study project](https://github.com/achliopa/udemy_BeginnersROSCourse/blob/master/readme.md) installation instructions (note these are for x64 machine)
* we consult [JetsonHacks Git](https://github.com/jetsonhacks/installROS) bash script for the our installed version of Jetpack (4.3) and [Official ARM ROS Install Instructions](http://wiki.ros.org/melodic/Installation/Ubuntu)
* Steps to take:
	* Setup the machine to accept SW from ROS `sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'`
	* Setup our keys `sudo apt-key adv --keyserver 'hkp://keyserver.ubuntu.com:80' --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654`
	* Note that keys change from time to time for security reasons. visit above site to check if needed
	* make sure our debian package index is upt to date `sudo apt update`
	* we use full install as we we ll do testing on the target (we dont have a powerful host yet `sudo apt install ros-melodic-desktop-full` in later stages for production we might have to go for `sudo apt install ros-melodic-ros-base`
	* to install an individual package we use `sudo apt install ros-melodic-PACKAGE` useful for later on if we use eg 'slam-gmapping'
	* initialize rosdep
```
sudo rosdep init
rosdep update
```
	* add ROS env vars to bash session
```
echo "source /opt/ros/melodic/setup.bash" >> ~/.bashrc
```
	* what we have installed so far is enough to run ROS packages. if we want to create and manage our own ROS packages on the device we need more tools. note that we do this as we want to do full development on the device. when we have a suitable host we should not do this step (probably) in the concept of barebone installation
```
sudo apt install python-rosinstall python-rosinstall-generator python-wstool build-essential
```

## Run our first Test ROS App on Tegra

* we will follow steps from our [Beginners ROS course notes](https://github.com/achliopa/udemy_BeginnersROSCourse/blob/master/readme.md)
* we lauch a ROS master `roscore`
* we create a catkin workspace in our workspace. this is where our projects will live
```
mkdir catkin_ws
cd catkin_ws
mkdir src
catkin_make
```
* we need to `source <PATH>/catkin_ws/devel/setup.bash` to carry on building projects
* we add it to `~/.bashrc` and restart terminal
* Create a Package
	* we create a new package: in `/catkin_ws/src` we run `catkin_create_pkg my_first_package roscpp rospy std_msgs`
	* we build the package: we go to `/catkin_ws` and run `catkin_make`
* Create a Python Node
	* we go in our workspace `/catkin_ws/src/my_first_package`
	* we create a scripts folder `mkdir scripts` and cd into it
	* we add a python script and make it executable
```
touch my_first_node.py
chmod +x my_first_node.py
```
	* we implement the code (it just writes to stdout)
```
#!/usr/bin/env python

import rospy

if __name__ == '__main__':
	rospy.init_node('my_first_python_node')

	rospy.loginfo("This node has been initialized")

	rate = rospy.Rate(10)

	while not rospy.is_shutdown():
		rospy.loginfo('Hello')
		rate.sleep()
	rospy.loginfo("Exit now")
```
	* we run the script with `python my_first_node.py` and see the output inthe console. we stop it with ctrl+C
	* success. (we need to have roscore running to run it succesfully
	* with our node running we can see it in `rosnode list`
	* we stop roscore (ctrl+C)
