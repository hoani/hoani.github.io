---
title: "ROS CLI cheatsheet"
excerpt: "A bunch of CLI commands for ROS"
toc: true
permalink: /guides/engineering/ros/ros1
categories:
  - guide
  - software
  - software-guide
  - ros-guide
---

### Environment Setup

I add the following aliases to my `.bashrc` (in Linux):
```bash
export ROS_CATKIN_PATH=~/catkin_ws
export ROS_NOETIC_SETUP=/opt/ros/noetic/setup.bash

alias rosnoetic='source $ROS_NOETIC_SETUP; source $ROS_CATKIN_PATH/devel/setup.bash'
alias rosbuild='cd $ROS_CATKIN_PATH; catkin_make; cd -'
```

When I need ros noetic avaliable from the command line, I run `rosnoetic`.

When I want to rebuild my entire catkin workspace, I run `rosbuild`.


### Navigation

```sh
roscd # Go to distro root
roscd <package> # Go to <package> folder
rosls <package> # List files in <package>
```

### Making Packages with Catkin

See [Creating a workspace for catkin](http://wiki.ros.org/catkin/Tutorials/create_a_workspace)

General structure:

```
catkin_ws/
  src/
    CMakeLists.txt 
    package_1/
      CMakeLists.txt
      package.xml
      src/
      scripts/
      msg/
      srv/
      ...
    package_2/
      ...
  devel/
    setup.bash
    ...
  build/
    ...
  .catkin_workspace
```

Catkin can be used to build and install python and cpp packages. 
See the [creating a package tutorial](http://wiki.ros.org/ROS/Tutorials/CreatingPackage).

### ROS Master

```sh
roscore
```

### Nodes

Run a `<pkg>::<node>`:

```sh
rosrun <pkg> <node>
rosrun turtlesim turtlesim_node # Run turtlebot simulator.
```

Analyze running nodes:

```sh
rosnode list        # Show running nodes
rosnode info <node> # Show info on particular node
rosnode ping <node> # Ping your node
```

### Topics

Analyzing topics:

```sh
rostopic list         # Show active topics
rostopic list -v      # Show active topics with types/stats
rostopic bw <topic>   # Show bandwidth of <topic>
rostopic type <topic> # Show the type name of <topic>
```

Subscribing to a topic from the command line:

```sh
rostopic echo <topic> # Subscribe to and echo <topic>
```

Publishing a topic:

```sh
rostopic pub <topic> <type> -- <data>
rostopic pub <topic> <type> -r <Hz> -- <data>

rostopic pub /hello /std_msgs/String -- "hello"
rostopic pub /hello /std_msgs/String -r 2 -- "hello @ 2Hz"
```

Publishing more complex structures uses YAML:
```sh
rostopic pub /turtle1/cmd_vel geometry_msgs/Twist -r 1 -- \
  '{linear: {x: 2.0, y: 0.0, z: 0.0},
  angular: {x: 0.0,y: 0.0,z: 0.0}}'
```

Topic types can be analyzed using `rosmsg`:
```sh
> rosmsg show geometery_msgs/Twist
geometry_msgs/Vector3 linear
  float64 x
  float64 y
  float64 z
geometry_msgs/Vector3 angular
  float64 x
  float64 y
  float64 z
```

### Services

Analyzing services:
```sh
rosservice list # Show all active services
rosservice info <service> # Get URI, type and args for <service>
rosservice uri <service> # Get <service> uri
rosservice type <service> # Get <service> type
rosservice args <service> # Get <service> args
```

Calling a service from CLI:
```sh
rosservice call <service> <args>

rosservice call /rosout/set_logger_level ros debug
```

Ros Service types are analyzed using `rossrv`:
```sh
> rossrv show turtlesim/Spawn
float32 x
float32 y
float32 theta
string name
---
string name
```

### Params

Get/set the value of a `<param>`
```sh
rosparam get <param>
rosparam set <param> <value>
```

### Launch files

Launch files allow us to create a consistent launch setup for nodes.

See the [launch file tutorial](http://wiki.ros.org/ROS/Tutorials/UsingRqtconsoleRoslaunch#Using_roslaunch) for details on writing a launch file.

To run a launch file from a ROS package:

```sh
roslaunch <package> <file.launch>
```

### Bag - data recording

Recording:
```sh
rosbag record -a                # Everything
rosbag record <topic1> <topic2> # Only topics 1 and 2
rosbag record -e <regex>        # Topics matching regex
rosbag record -a -O <name>      # Name the bag
rosbag record -h                # Show record help
```

Get information about bag `<name>`:
```sh
rosbag info <name>
```

Play the information in bag `<name>`:
```sh
rosbag play <name>
rosbag play <name> -l # Loop forever.
```
