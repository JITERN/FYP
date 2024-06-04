## Overview

This repo documents how RTAB-Map can be used to compare the mapping and localization performance of various sensor combinations on a robotic wheelchair.

**Maintainer:** [Jit Ern](limjitern@gmail.com)

> *This package has been tested under ROS Noetic on Ubuntu 20.04.*

---

## Installation

### Dependencies

- RTAB-Map installed and built from source with -DRTABMAP_SYNC_MULTI_RGBD=ON added to catkin_make, official installation instructions: https://github.com/introlab/rtabmap_ros?tab=readme-ov-file#build-from-source
- Python evo package to evaluate the localization accuracy: https://github.com/MichaelGrupp/evo.git
  
### Building

In a catkin workspace
```bash
cd ~/catkin_ws/src
git clone https://github.com/JITERN/FYP.git
cd ..
catkin_make
source devel/setup.bash
```

## Usage

Parameters:
- num_cameras: 1 or 2
- scan: true or false
- cloud: true or false
- grid: 0,1 or 2 (Create occupancy grid from selected sensor: 0=laser scan, 1=depth image(s) or 2=both laser scan and depth image(s).)
- mapping: true or false (defaults to false = localization only mode)

Note: scan and cloud cannot be set to true at the same time.

### Mapping 
Example with 3DLidar+2RGBD using both sensors for grid creation
```bash
roslaunch my_robot_navigation rtabmap.launch  num_cameras:=2  scan:=false  cloud:=true  grid_sensor:=2  mapping:=true
```

To clear the rtabmap database and restart mapping from scratch, click "Edit>Delete memory" in RTAB-Map Viz window. Always click "Detection>Trigger a new map" on every seperate mapping session.

Play a rosbag or drive the robot in real-time, CTRL-C in the terminal where RTAB-Map is running to end the session and the session will be automatically saved.
```bash
rosbag play --clock lab.bag
```

To export the 2D occupancy grid
```bash
#rtabmap-databaseViewer <path_to_rtabmap.db>
rtabmap-databaseViewer .ros/rtabmap.db 
```
Click "File>Regenerate optimized 2D map", then "File>Export optimized 2D map"

### Localization
Example with 2DLidar+1RGBD
```bash
roslaunch my_robot_navigation rtabmap.launch  num_cameras:=1  scan:=true  cloud:=false  grid_sensor:=2 
```

Start roscore
```bash
roscore
```
On another terminal, record new rosbag with only TF data for localization trajectory evaluation
```bash
rosparam set use_sim_time true
rosbag record /tf /tf_static
```

```bash
rosbag play --clock lab.bag
```

Use Python evo package to process the new rosbag
