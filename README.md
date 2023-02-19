# Elephant Train Collision Prevention System Developer Guide
---
<br>

## **Introduction**
<br>

The main objective of developing this system is to provide the train drivers with a monitoring interface for train drivers and other interested parties, so that they are able to take necessary action when some threats are identified. Threat in the sense, means something harmful to the elephants. For an example in Sri Lanka, the collisions between elephants and trains have been a huge problem that needs fixing. By using this platform threats like elephants' presence in the railtrack and elephants' moving directions can be easily monitored and then necessary actions can be taken to reduce the potential damage that can be caused by issues.

There are few modules to perform the complete task that the system intends to perform and the components are connected with Robot Operating System.  
<br>
<br>

## **System Components**
<br>

* ### **Object Detection Component**  

This is Powered by a Yolov3 engine, that contains a ros-node with Darknet implementation. This is based on the [Darknet-ROS](https://github.com/leggedrobotics/darknet_ros) ros package by *leggedrobotics*. For using a custom detection model a yolov3 model should be trained with a custom dataset and put inside this package. (Full configuration details can be found in Setting Up section.)

* ### **Video Streaming Component**
This Node takes the video stream from a source (in this case, from an IP camera) and publishes it to a topic called ```/img-raw```. The whole purpose of this node is to make the video stream accessible to the other nodes of the system.

* ### **Collision Detection Component** 
After detecting the objects it is required to track each one for predicting the location and the movement directions of them. This data comes handy when we are trying to predict a threat event like elephant is moving towards the railtrack etc. The object tracking is performed using the dimensions of bounding boxes identified by the object detection node and comparing similarities between the objects detected by object detection node.

* ### **GUI Component**
While detections and predictions are handled the results should be displayed realtime. For that a simple GUI is developed as a ros-package (developed using pyqt5 python library)  
<br>
<br>

## **Details of Nodes in each components and their topic details**
<br>

* ### **Object Detection Component**  
All the details could be find using the [official documentation](https://github.com/leggedrobotics/darknet_ros).  


* ### **Video Streaming Component**  
The package is named as ```Reolink_read```  
```python
*Nodes*  
    -reolink_pub.py
    -reolink_sub.py


    *reolink_pub.py*
        -   /image-raw     # Topic for publishing the video stream

    *reolink_sub.py*
        #Includes a topic for subscribing to /image_raw topic. (Not used in the system. Implemented only for debugging purposes.)  
```  

* ### **Collision Detection Component**
The package is named as ```col_det_pkg_v2```
```python
    *Nodes*
        -col_det_node.py

        *col_det_node.py
            -   /threats     # Publishing topic, Publishes data about the threat events and snapshots.
            -   /image-raw
            -   /bbox        # Subscribed to this topic to extract the bounding box details. (This topic is published by darknet-ros component.)
```
<br>
<br>

## **Preparation of Linux system for implementing the complete platform**
<br>

This step can vary on the fact that you are using an PC that installed Ubuntu or a Jetson board. If you are using a PC which Ubuntu is installed, the setup can take longer since we have to refer the official sites and detailed steps for setting up those things. But in case of using Jetson boards, there are pre-configured scripts. Also when it comes to setting up CUDA, the [SDK-Manager](https://developer.nvidia.com/drive/sdk-manager) provides a convenient way to set it up and therefore in case of jetson boards, it is much easier.

### **Ubuntu PC**

- Setup CUDA and CUDNN  in the machine.  
    * https://docs.nvidia.com/cuda/cuda-installation-guide-linux/
    * https://docs.nvidia.com/deeplearning/cudnn/install-guide/index.html (recommanded article)

- Installing OpenCV from source  
    * https://docs.opencv.org/3.4/d2/de6/tutorial_py_setup_in_ubuntu.html
    * https://linuxize.com/post/how-to-install-opencv-on-ubuntu-20-04/
- Installing ROS on ubuntu  
    * http://wiki.ros.org/Documentation  

Note - When you are installing OpenCV from source there are someflags for setting cuda related things and python related settings (like choosing python interpreter and the python library path etc.). Make sure you have set them corectly in CMAKE settings.(You can use CMAKE-gui for easier visualization). In official doc of OpenCV, there's no mention about installing opencv-contrib with opencv but that is crucial. Refer the linuxize guide to understand about that.  


### **Jetson AGX**

- Install ubuntu on a PC and install [SDK-MANAGER](https://developer.nvidia.com/drive/sdk-manager) in it.
- Connect jetson board to that PC and make sure it is visible in SDK-Manager
- Install the jetpack version you desire.
- After installation is complete, connect it again to the coomputer and install latest CUDA and CUDNN versions which are compatible with the jetson gpu architecture.
- Install opencv on the jetson using the script provided by *jetson-hacks*.
- Install ROS as instructed on the official docs.  
    - https://github.com/JetsonHacksNano/buildOpenCV  (remember to change the ARCH_BIN flag on .sh file to match the jetson board.)  
<br>
<br>
<br>

## **Implementing the ROS workspace and Packages**
<br>

Once you have setup all the pre-requisits now you can implement the system inside it.
1. First create a ros workspace in a desired directory, and create a workspace in it.
```
mkdir -p ws/src
```

2. Create a directory (outside the workspace, better called ros-pkgs) to save the ros packages. And copy all the ros packages related to project in there.
```
mkdir ros-pkgs
```

3. Now create symlinks in ```ws/src``` referencing to all the packages inside the ros-pkgs directory.

```python
ln -s ../../ros-pkgs/Reolink_read .  # creating a symlink for the package Reolink_read
```

4. If you are using ros-melodic version, you will have to clone the ```visions_opencv``` package to help the cv_bridge to function properly.
```
https://github.com/ros-perception/vision_opencv    # (clone this)
After cloning,
    Go to the ros workspace directory.
    catkin config --install
    catkin build
```
If everything went right, you have successfully implemented the project in your system.  
Refer [setting up cv_bridge](https://medium.com/@beta_b0t/how-to-setup-ros-with-python-3-44a69ca36674) if you run in to an issue with ```visions_opencv``` ros package
<br>
<br>
<br>

## **Running the program**
<br>

Since there are few components in the project either it is possible to run the program by activating each node seperately or it could be run by activating all the nodes using roslaunch file that is configured in the GUI package.

1. Activating each node at a time
```python
#start the darknet-ros nodes(and wait till it completes initiating)
roslaunch darknet-ros yolov3.launch

#start the collision detection node and gui nodes (the order you activate them are not crucial)
rosrun col_det_pkgv2 collision_detection_node_v1.py 
rosrun gui_pkgv2 gui_final.py


#finally activate the node for video stream.
rosrun reolink_read reolink_pub.py
```  
2. Activating all the nodes at once.
```
roslaunch gui_pkgv2 launch_file.launch
```
Few points to note -  
- You can change the video source by editing the input (video source) of the ```cv2.VideoCapture```, and for testing purposes you can use a video file for that.
- Since it is stil in the developing state a **cronjob** for starting the program at the startup of the PC or jetson board has not been set yet, but in the production it is better if you can set one.
