# Elephant Train Collision Prevention System Developer Guide
---
<br>

## Introduction
<br>

The main objective of developing this system is to provide the train drivers with a monitoring interface for train drivers and other interested parties, so that they are able to take necessary action when some threats are identified. Threat in the sense, means something harmful to the elephants. For an example in Sri Lanka, the collisions between elephants and trains have been a huge problem that needs fixing. By using this platform threats like elephants' presence in the railtrack and elephants' moving directions can be easily monitored and then necessary actions can be taken to reduce the potential damage that can be caused by issues.

There are few modules to perform the complete task that the system intends to perform and the components are connected with Robot Operating System.  
<br>

## System Components
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

## Details of Nodes in each components and their topic details.  
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
        -/image-raw     # Topic for publishing the video stream

    *reolink_sub.py*
        #Includes a topic for subscribing to /image_raw topic. (Not used in the system. Implemented only for debugging purposes.)  
```  

* ### **Collision Detection Component**
The package is named as ```col_det_pkg_v2```
```python
    *Nodes*
        -col_det_node.py

        *col_det_node.py
            -/threats     # Topic for publishing the threat events and snapshots.

