tutorial_rosbridge_ardrone
==========================

Front-end web interface code associated with teleoperation tutorial for AR.Drone using rosbridge/ROS.  This is a simple interface to illustate basic concepts, and not a maintained release.  Please refer to robotwebtools.org for the latest and greatest.


AR.Drone and ROS/rosbridge Tutorial

This tutorial covers how to:

    control an AR.Drone using the Robot Operating System (ROS), and
    create a web browser interface for the AR.Drone using rosbridge

This tutorial assumes an AR.Drone v1.0 and separate computer running Ubuntu 12.04 with root privileges, wifi access, and current installations of git, svn, make, and cmake.
AR.Drone Control through ROS

    Install the Fuerte distribution of ROS (ros.org)
    Create the working directory for this tutorial (”~/ros”):

mkdir ~/ros
cd ~/ros

    Ensure the working directory is included in the ROS_PACKAGE_PATH environment variable. This can be done using the export command:

export ROS_PACKAGE_PATH=$ROS_PACKAGE_PATH:/home/willow/ros

    Install the ardrone_autonomy package. This is the ROS driver for the AR.Drone, which additionally requires building the AR.Drone SDK:

cd ~/ros
git clone https://github.com/AutonomyLab/ardrone_autonomy.git
cd ardrone_autonomy
./build_sdk.sh
cmake .
rosmake ardrone_autonomy

    Install the drone_teleop package

cd ~/ros
svn checkout http://brown-ros-pkg.googlecode.com/svn/trunk/experimental/drone_teleop

    Power the drone and wait until its indicator lights turn green
    Connect the computer to the drone's network, e.g., SSID “ardrone_123456”)
    Run the ROS master and drone driver in one terminal:

roscore 
rosrun ardrone_autonomy ardrone_driver

    Run the drone control client in another terminal:

rosrun drone_teleop drone_teleop.py

If successful, you should be able to takeoff, drive, and and land the drone from the instructions in the terminal.
In-Browser AR.Drone Control through rosbridge

The following assumes roscore and ardrone_autonomy (described above) are still running.

    Install rosbridge and mjpeg-server:

sudo apt-get install ros-fuerte-rosbridge-suite
sudo apt-get install ros-fuerte-mjpeg-server

    This will allow this web page to be served using roswww:

cd ~/ros/drone_teleop
mkdir www
cd www
wget https://dl.dropbox.com/u/14391589/tutorial/rosbridge_ardrone/drone_browser_teleop.html
wget https://dl.dropbox.com/u/14391589/tutorial/rosbridge_ardrone/ros.js

    As an alternative, you can also clone this web interface and another interface with buttons from this github repository.

    Run rosbridge, mjpeg_server, and roswww:

rosrun rosbridge_server rosbridge.py
rosrun mjpeg_server mjpeg_server
rosrun roswww webserver.py

    Navigate a websocket-enabled browser to http://localhost:8000/drone_teleop/drone_browser_teleop.html

If successful, you browser should look like the following image, with a live video stream from the drone's front camera. You should now be able to takeoff, drive, and land the drone through the in-browser instructions.

How It Works

drone_browser_teleop.html works by using ros.js to connect to ROS via rosbridge, where it can then send the same messages as the drone_teleop ROS node to control the drone. The video stream is served by mjpeg_server through an img tage. The following go through this in more detail.

drone_browser_teleop.html includes the file “ros.js” as a JavaScript interface to rosbridge. There are many different ros.js clients. This one was taken from the “rosbridge_clients” examples included with the rosbridge_suite install:

<script type="text/javascript" src="ros.js"></script>

The instantiation of a Bridge object from ros.js creates a websocket connection to rosbridge, which runs by default on port 9090:

var con = new Bridge("ws://localhost:9090");

Once the Bridge object is created, the web page can publish and subscribe messages to ROS, just like any other ROS node. In the drone teleop example, communication with the drone requires three particular messages for takeoff, landing, and sending a “twist” 6 degree-of-freedom velocity, respectively:

// Takeoff
con.publish('/ardrone/takeoff', { });

// Land
con.publish('/ardrone/land', { });

// Forward
con.publish('/cmd_vel', {"linear":{"x":1.0,"y":0,"z":0},"angular":{"x":0,"y":0,"z":0}});
// Left
con.publish('/cmd_vel', {"linear":{"x":0,"y":0,"z":0},"angular":{"x":0,"y":0,"z":-1.0}});

The cameras video stream is provided easily by creating an image tag to the ”/ardrone/image_raw” topic, which is served by mjpeg_server by default on port 8080:

<img src="http://localhost:8080/stream?topic=/ardrone/image_raw">

This is not an actual ROS subscription, but a special feature provided by mjpeg_server. Proper subscriptions to any topic in the ROS run-time can be accessed through ros.js. Please examine ros.js for further details.

Happy flying, happy hacking.
