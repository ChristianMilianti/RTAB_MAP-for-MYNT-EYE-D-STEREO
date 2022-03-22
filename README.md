

# RTAB MAP for MYNT EYE D STEREO
Repository for using the MYNT EYE D Stereo Camera with RTAB MAP in ROS (Currently only supports MYNT S Stereo Camera) for handheld stereo mapping 

The following assumes you have basic familiarity with ROS, have a catkin_ws and are familiar with catkin-tools


![GitHub Logo](/field_test1_isometric.png)


# Prerequisites
Installation of ROS Melodic 

LINK: http://wiki.ros.org/melodic/Installation/Ubuntu

Source Install of rtabmap_ros

LINK: https://github.com/introlab/rtabmap_ros

Installation of the MYNT EYE D SDK and the mynt eye ros wrapper

LINK: https://github.com/slightech/MYNT-EYE-D-SDK

# The following Setup can be completed manually, otherwise the edited files are provided for your convenience
Simply copy files into the directories descirbed below


# Setup of MYNT-EYE-D-SDK for rtabmap_ros

in the [rtabmap_ros wiki](http://wiki.ros.org/rtabmap_ros/Tutorials/StereoHandHeldMapping#Note) there is mention of an issue wih right camera info (for the mynt eye s stereo varian) value which is based in mm instead of meters. 

This issue persists in the MYNT EYE D SDK aswell, to solve this issue open the mynteye_wrapper_nodelet.cc from the following path: ~/MYNT-EYE-D-SDK/wrappers/ros/src/mynteye_wrapper_d/src/mynteye_wrapper_nodelet.cc
 and edit line 905 from:
 camera_info->P.at(3) = in.p[3] 
 to 
 camera_info->P.at(3) = in.p[3]/1000;  

follow the instruction in the provided ros wiki link to test solution.

Additionally RTAB_MAP requires rectified images to work to get the camera to output rectified images do the following

in the MYNT-EYE-D-SDK open get_images.cc from the following path:

path ~/MYNT-EYE-D-SDK/samples/src/get_image.cc

and uncomment the following line:

params.color_mode = ColorMode::COLOR_RECTIFIED;

open the terminal and run "make samples" to build 

finally put the provided "RTAB_mynteye.launch" file into the following directory:

Path: ~/MYNT-EYE-D-SDK/wrappers/ros/src/mynteye_wrapper_d/launch/RTAB_mynteye.launch


# Replace the provided launch file with the one in the launch folder of the rtabmap_ros package
Copy the provided file to the following directory This is an edited version of the launch file customised to suit the mynt stereo camera

~/catkin_ws/src/rtabmap_ros/launch/rtabmap.launch

# Usage 
Open three termnial windows 

Terminal 1: 
roscore

Terminal 2:
cd Documents/MYNT-EYE-D-SDK -run this first

 source ./wrappers/ros/devel/setup.bash - run this second

  cd wrappers/ros/src/mynteye_wrapper_d/launch/ -run this third

roslaunch RTAB_mynteye.launch - run this fourth, (this 'turns on the camera' and starts publishing the topics depending on what is specified within the RTAB_mynteye.launch file.)


If you want to do a shortcut use the following: cd Documents/MYNT-EYE-D-SDK && source ./wrappers/ros/devel/setup.bash && roslaunch mynteye_wrapper_d RTAB_mynteye_backup.launch

Terminal 3:
These next steps launch rtabmap, with the required parameters in order to work with the data being published by the mynt eye camera.

Run this for colour:

roslaunch rtabmap_ros rtabmap.launch    stereo:=true    left_image_topic:=/mynteye/left_rect/image_rect_color    right_image_topic:=/mynteye/right_rect/image_rect_color    left_camera_info_topic:=/mynteye/left_rect/camera_info    right_camera_info_topic:=/mynteye/right_rect/camera_info    frame_id:=mynteye_link_frame    rtabmap_args:="-d"
   
  OR this for monochrome SLAM
  
  roslaunch rtabmap_ros rtabmap.launch \
   stereo:=true \
   left_image_topic:=/mynteye/left_rect/image_rect \
   right_image_topic:=/mynteye/right_rect/image_rect \
   left_camera_info_topic:=/mynteye/left_rect/camera_info \
   right_camera_info_topic:=/mynteye/right_rect/camera_info \
   frame_id:=mynteye_link_frame \
   rtabmap_args:="-d"



# For 2022 team: use the following ros instructions

roscore

------NEW TERMINAL TAB-----

cd Documents/MYNT-EYE-D-SDK && source ./wrappers/ros/devel/setup.bash && roslaunch mynteye_wrapper_d RTAB_mynteye_backup.launch

------NEW TERMINAL TAB-----

1877  sudo chmod 666 /dev/ttyUSB0
and then write rmit as the password

roslaunch sbg_driver sbg_ellipseN.launch

------NEW TERMINAL TAB-----

roslaunch darknet_ros_3d darknet_ros_3d_beijing.launch

------NEW TERMINAL TAB-----

roslaunch darknet_ros_3d darknet_ros_3d.launch

------NEW TERMINAL TAB-----

roslaunch rtabmap_ros rtabmap.launch stereo:=true left_image_topic:=/mynteye/left_rect/image_rect_color right_image_topic:=/mynteye/right_rect/image_rect_color left_camera_info_topic:=/mynteye/left_rect/camera_info right_camera_info_topic:=/mynteye/right_rect/camera_info frame_id:=mynteye_link_frame gps_topic:=/imu/nav_sat_fix rtabmap_args:="-d"

------NEW TERMINAL TAB-----

rosbag record rosout tf /darknet_ros_3d/bounding_boxes /mynteye/left_rect/image_rect_color /rosout_agg /imu/data /imu/nav_sat_fix /rtabmap/grid_map /rtabmap/odom /rtabmap/odom_info /rtabmap/cloud_ground /rtabmap/cloud_map /rtabmap/cloud_obstacles

--- Go to MATLAB ------

then go to matlab and run the map generation script (note, there are a few different scripts available to run, but i believe the bottom one is the most recent one which also gives the acceleration graph vs time which i think is accuratish)

"map_generator_universal_efficient_withimudata" - this script can be found in /home/rmitdriverless/Documents/ChristianMilianti/MATLAB code from 2021 team_backedtossd/map_generator_universal_efficient_withimudata.m

remember to run 'rosshutdown' in matlab command line when the rosbag has finished running.




