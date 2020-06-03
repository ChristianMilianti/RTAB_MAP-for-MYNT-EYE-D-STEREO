

# RTAB_MAP-for-MYNT-EYE-D-STEREO
Repository for using the MYNT EYE D Stereo Camera with RTAB MAP in ROS (Currently only supports MYNT S Stereo Camera) for handheld stereo mapping 

Tutorial assumes you have basic familiarity with ROS, have a catkin_ws and are familiar with catkin-tools

# Prerequisites
Installation of ROS Melodic 

LINK: http://wiki.ros.org/melodic/Installation/Ubuntu

Source Install of rtabmap_ros

LINK: https://github.com/introlab/rtabmap_ros

Install of the MYNT EYE D SDK and build the mynt eye ros wrapper

LINK: https://github.com/slightech/MYNT-EYE-D-SDK


# Setup of MYNT-EYE-D-SDK for rtabmap_ros (REQUIRED)

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


# Add the provided launch file to rtabmap_ros package (REQUIRED)
Copy the provided file to the following directory 

~/catkin_ws/src/rtabmap_ros/launch/rtabmap_mynt.launch

# Usage 
Open three termnial windows 

Terminal 1: 
roscore

Terminal 2:
cd MYNT-EYE-D-SDK
source ./wrappers/ros/devel/setup.bash
cd ~/MYNT-EYE-D-SDK/wrappers/ros/src/mynteye_wrapper_d/launch/

roslaunch RTAB_mynteye.launch

Terminal 3:

Run this for colour:

roslaunch rtabmap_ros rtabmap_mynt.launch \
   stereo:=true \
   left_image_topic:=/mynteye/left_rect/image_rect_color \
   right_image_topic:=/mynteye/right_rect/image_rect_color \
   left_camera_info_topic:=/mynteye/left_rect/camera_info \
   right_camera_info_topic:=/mynteye/right_rect/camera_info \
   frame_id:=mynteye_link_frame \
   rtabmap_args:="-d"
   
  OR this for monochrome SLAM
  
  roslaunch rtabmap_ros rtabmap_mynt.launch \
   stereo:=true \
   left_image_topic:=/mynteye/left_rect/image_rect \
   right_image_topic:=/mynteye/right_rect/image_rect \
   left_camera_info_topic:=/mynteye/left_rect/camera_info \
   right_camera_info_topic:=/mynteye/right_rect/camera_info \
   frame_id:=mynteye_link_frame \
   rtabmap_args:="-d"


