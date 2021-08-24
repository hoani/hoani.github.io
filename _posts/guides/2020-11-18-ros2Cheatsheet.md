---
title: "ROS2 CLI cheatsheet"
excerpt: "A bunch of CLI commands for ROS2"
toc: true
categories:
  - guide
  - software
  - software-guide
  - electronics
  - electronics-guide
---

### Listing stuff

```sh
ros2 pkg executables <pkgname>
ros2 node info <node>
ros2 topic list [-t]
ros2 service list [-t]
ros2 action list [-t]
ros2 param list
```

Where `-t` will add the types in square brackets.

### Nodes

Run a `<pkg>::<executable>` node

```sh
ros2 run <pkg> <executable>
```

Set the `<log-level>` (`DEBUG`, `INFO`, `WARN`, `ERROR` or `FATAL`):

```sh
ros2 run <pkg> <executable> --ros-args --log-level <log-level>
```

### Topics

Subscribe to and echo `<topic-name>`:
```sh
ros2 topic echo <topic-name>
```

Get the type and number of pub/subs to `<topic-name>`:
```sh
ros2 topic info <topic-name>
```

See how often `<topic-name>` is being published:
```sh
ros2 topic hz <topic-name>
```

Check a topic's interface using the `<type>`:
```sh
ros2 interface show <type>
```

Publish to a topic:
```sh
ros2 topic pub <topic-name> <type> "<args>"
```

Where `"<args>"` are input in YAML syntax, ie: `"{battery: {voltage: 12.7, capacity: 50.0}}"`

Options:
* `--once` publishes the message once then exits
* `--rate <freq>` publishes the message at `<freq>` Hz

### Services

Get the type of `<service-name>`:
```sh
ros2 service type <service-name>
```

*Note: type `std_srvs/srv/Empty` sends and receives no data* 

Find services of a particular `<type>`:
```sh
ros2 service find <type>
```

Show a service's interface using the `<type>`:
```sh
ros2 interface show <type>
```

Which prints:

```
<request structure>
---
<reponse structure>
```

Call a service using:

```sh
ros2 service call <service-name> <type> "<args>"
```

Like topics, `"<args>"` are input in YAML syntax, ie: `"{gains: {kp: 1.0, ki: 0.0, kd: 0.1}}"`

### Actions

Information on `<action>` clients and servers:
```sh
ros2 action info /turtle1/rotate_absolute
```

Show an actions's interface using the `<type>`:
```sh
ros2 interface show <type>
```

Which prints:

```
<request structure>
---
<result structure>
---
<feedback structure>
```

Send a goal to an `<action>`:

```sh
ros2 action send_goal <action> <type> "<args>"
```

`"<args>"` are input in YAML syntax, ie: `"{setpoint: {x: 1.0, y: 0.0, z: 0.1}}"`


### Params

Get/set the value of a `<node>`:`<param>`
```sh
ros2 param get <node> <param>
ros2 param set <node> <param> <value>
```

Dumping and loading parameters:

```sh
ros2 param dump <node> # Saves params to <node>.yaml
ros2 run <pkg> <executable> \
    --ros-args --params-file <node>.yaml
```

### Bag - data recording

Record `<topics>`:
```sh
ros2 bag record <topic-0> [<topic-1>] ... [<topic-N>]
ros2 bag record -o <name> <topic-0> [<topic-1>] ... [<topic-N>]
```

Use the `-o` flag to set a custome bag `<name>`. The default is `rosbag2_yyyy_MM_dd-HH_mm_ss`.

Get information about bag `<name>`:
```sh
ros2 bag info <name>
```

Play the information in bag `<name>`:
```sh
ros2 bag play <name>
```

### Tools

`rqt_console` monitors log messages:
```sh
ros2 run rqt_console rqt_console
```

`rqt_graph` shows the current ROS graph:
```sh
rqt_graph
```

`launch` a bunch of nodes defined in a `<launch-script>.py`, see [launch tutorial](https://index.ros.org/doc/ros2/Tutorials/Launch-Files/Creating-Launch-Files/#ros2launch) for more details
```sh
ros2 launch <launch-script>.py
ros2 launch <pkg> <launch-script>.py
```

