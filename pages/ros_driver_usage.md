# ROS driver usage

The ROS driver can be started with

```
rosrun alphasense_driver_ros alphasense_driver_ros
```

## ROS API

### Published topics

Images are published using the `image_transport` package. The topics published depend on the `image_transport` plugins installed. We list only the raw image topics here.

* **/alphasense_driver_ros/cam0** (*sensor_msgs/Image*)  
  Camera images from cam0
* **/alphasense_driver_ros/cam1** (*sensor_msgs/Image*)  
  Camera images from cam1
* **/alphasense_driver_ros/cam2** (*sensor_msgs/Image*)  
  Camera images from cam2
* **/alphasense_driver_ros/cam3** (*sensor_msgs/Image*)  
  Camera images from cam3
* **/alphasense_driver_ros/cam4** (*sensor_msgs/Image*)  
  Camera images from cam4
* **/alphasense_driver_ros/cam5** (*sensor_msgs/Image*)  
  Camera images from cam5
* **/alphasense_driver_ros/cam6** (*sensor_msgs/Image*)  
  Camera images from cam6
* **/alphasense_driver_ros/cam7** (*sensor_msgs/Image*)  
  Camera images from cam7
* **/alphasense_driver_ros/imu** (*sensor_msgs/Imu*)  
  IMU measurements of the internal IMU.
* **/alphasense_driver_ros/aux_imu** (*sensor_msgs/Imu*)  
  IMU measurements of an optional external IMU.
* **/alphasense_driver_ros/serial_number** (*std_msgs/String*)  
  Serial number of the connected Core Research.
  
### Parameters

* **~ncamera_settings** (*string, default: ""*)  
  File path to a `ncamera_settings.yaml` configuration file. See [Sensor settings](/pages/sensor_settings.md) for more information about the format of this file. If left empty `rqt_reconfigure` support will be enabled, configuration can then be changed dynamically and is read from the ros parameter system.
* **~device_serial** (*string, default: ""*)  
  When this is set the driver will only connect to a specific Core Research. If left empty the driver will connect to the first one it can find.
* **~translate_device_time** (*bool, default: True*)  
  Translate the internal Core Research clock to the host clock using [`cuckoo_time_translator`](https://github.com/ethz-asl/cuckoo_time_translator). This should be disabled when the internal Core Research clock is synchronized to the host using PTP (see [Synchronized time](/pages/time_synchronization.md#synchronized-time-ptp)).
