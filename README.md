

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

roslaunch RTAB_mynteye.launch - run this fourth, (this 'turns on the camera' and starts publishing the topics depending on what is specified within the RTAB_mynteye.launch file. - make sure the camera is plugged in)


If you want to do a shortcut use the following: cd Documents/MYNT-EYE-D-SDK && source ./wrappers/ros/devel/setup.bash && roslaunch mynteye_wrapper_d RTAB_mynteye_backup.launch

Terminal 3: Launch the SBG Driver - make sure the sbg is plugged in...


roslaunch sbg_driver sbg_ellipseN.launch

if driver fails to launch - do this sudo chmod 666 /dev/ttyUSB0


Terminal 4:
These next steps launch rtabmap, with the required parameters in order to work with the data being published by the mynt eye camera and sbg.

   
Run this for colour and SBG:
roslaunch rtabmap_ros rtabmap.launch stereo:=true left_image_topic:=/mynteye/left_rect/image_rect_color right_image_topic:=/mynteye/right_rect/image_rect_color left_camera_info_topic:=/mynteye/left_rect/camera_info right_camera_info_topic:=/mynteye/right_rect/camera_info frame_id:=mynteye_link_frame gps_topic:=/imu/nav_sat_fix rtabmap_args:="-d"


to record data:
rosbag record rosout tf /darknet_ros_3d/bounding_boxes /darknet_ros_3d/markers /mynteye/depth/camera_info /mynteye/depth/image_raw /mynteye/imu/data_raw /mynteye/points/data_raw /mynteye/right_rect/image_rect_color /mynteye/right_rect/camera_info /mynteye/left_rect/camera_info /mynteye/left_rect/image_rect_color /mynteye/temp/data_raw /rosout /rosout_agg /imu/data /imu/nav_sat_fix /rtabmap/grid_map /rtabmap/grid_prob_map /rtabmap/odom /rtabmap/odom_info /rtabmap/cloud_ground /rtabmap/cloud_map /rtabmap/cloud_obstacles /rtabmap/proj_map /rtabmap/scan_map

if you want it to be less resource heavy- try the following

rosbag record rosout tf /darknet_ros_3d/bounding_boxes /mynteye/left_rect/image_rect_color /rosout_agg /imu/data /imu/nav_sat_fix /rtabmap/grid_map  /rtabmap/odom /rtabmap/odom_info /rtabmap/cloud_ground /rtabmap/cloud_map /rtabmap/cloud_obstacles 


