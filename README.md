# Super Mega Bot Path Planner

Package for path planning of Super Mega Bots for the ETH Robotics Summer School. 
The package has been tested under ROS Melodic and Ubuntu 18.04.

__Author__: Luca Bartolomei  
__Affiliation__: Vision For Robotics Lab, ETH Zurich  
__Contact__: Luca Bartolomei, lbartolomei@ethz.ch  

# Installation instructions  
Install the following packages first:
```
sudo apt-get install ros-melodic-cmake-modules ros-melodic-velodyne-gazebo-plugins python-wstool python-catkin-tools ros-melodic-ompl ros-melodic-move-base ros-melodic-navfn ros-melodic-dwa-local-planner ros-melodic-costmap-2d ros-melodic-teb-local-planner ros-melodic-robot-self-filter ros-melodic-pointcloud-to-laserscan
```
Create a catkin workspace:  
```
mdkir -p ~/catkin_ws/src
cd ~catkin_ws
source /opt/ros/melodic/setup.bash
catkin init
catkin config --extend /opt/ros/melodic
catkin config --cmake-args -DCMAKE_BUILD_TYPE=Release
catkin config --merge-devel
```
Then, clone the repository in your catkin workspace:  
```
cd ~/catkin_ws/src
git clone git@github.com:VIS4ROB-lab/smb_path_planner.git
cd ~/catkin_ws
catkin build smb_path_planner
```  

## Additional documentation
Additional documentation on `move_base` can be found [here](https://wiki.ros.org/move_base), while for the local planner, refer to the TEB Local Planner instructions [here](https://wiki.ros.org/teb_local_planner).

## Structure of the code
The `smb_path_planner` package is composed by 3 packages:
* `smb_ompl_planner`: global planner for `move_base` based on the [OMPL library](http://ompl.kavrakilab.org/) for motion planning;
* `smb_navigation`: package containing utilities, configurations and launch files for the planning with `move_base`;
* `smb_navigation_rviz`: package containing the RViz plugin to put a goal for the planner easily;
* `traversability_layer`: custom `costmap_2d` implementation to incorporate traversability maps.

## Planning Panel in RViz
Make sure all the packages have built successfully. As a sanity check, re-source your workspace (`$ source ~/catkin_ws/devel/setup.bash`) and start up RViz (`$ rviz`). In RViz, select `Panels -> Add New Panel` and select `Planning Panel` under `smb_navigation_rviz`.

Next, under `Displays`, add an `InteractiveMarkers` display with the topic `/planning_markers/update`. You should be able to see the interactive markers and the planning panel.

## How to run the planner in simulation
First, start the simulation in Gazebo, RViz and RQT Plugin to select the 
controller, by running:
```
$ roslaunch smb_sim sim_path_planner.launch
```
In the controller panel, select `MpcTrackLocalPlan` from the list. If this controller does not show up, press the refresh button and try again. To start the controller, press the play button. Finally, start the local and global planners. If you want to use RRTs as global planner, run:
```
$ roslaunch smb_navigation navigate2d_ompl.launch
```  
Otherwise, to use a standard global planner from `move_base`, run:
```
$ roslaunch smb_navigation navigate2d.launch
```  
To send a global goal position there a set of different possibilities:
* Set a goal with the planning panel and press the button `Start Planning`;
* Use RViz direcly by using the button `2D Nav Goal` and setting the goal pose;
* Publish directly on the topic `/move_base_simple/goal`.  

### Running with traversability estimation
Start the simulation as in the previous case, and then run:
```
$ roslaunch smb_navigation navigate2d_ompl.launch run_traversability:=true
```
Notice that in this case, there are 3 different cost layers (traversability, laser scans and inflation layers), but **only** the traversability layer is active. To de-activate it, or to enable the other layers, run `rosrun rqt_reconfigure rqt_reconfigure` and enable/disable the layers.  

Notice that it is not possible to run the obstacle layer (based on laser scans) and the traversability layer at the same time in the current configuration, as the laser scan clears the traversability map. You are more than welcome to find a proper way to fuse these two maps!

## How to run the planner on the real robot
__THIS PART HAS TO BE UPDATED__
Start the `LPC` on the robot and the `OPC` on the operator side. To run the 
`LPC`, run the following commands in the PC of the robot in multiple terminals:
```
$ roscore # terminal 1
$ roslaunch smb_lpc lpc.launch # terminal 2
```
Start the LiDAR and the mapping in another terminal in the robot PC:
```
$ roslaunch ethzasl_icp_mapper supermegabot_robosense_dynamic_mapper.launch # terminal 3
``` 
 Then to start the planners, run on the PC of the robot in a separate terminal:
```
$ roslaunch smb_navigation navigate2d_ompl.launch # terminal 5
```
On the operator side, start the `OPC`. First connect the `rosmaster` of the 
PC to the `rosmaster` of the robot and then run in a terminal:
```
$ roslaunch smb_opc opc.launch
```
To send a global goal position, follow the same procedure as in simulation:
1. Select the `MpcTrackLocalPlan` in the control panel and start it;
2. Send the global goal via RViz or via terminal.  

