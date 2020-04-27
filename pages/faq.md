# Frequently asked questions

We would like to answer frequently asked questions on this page. If you do not
find an answer here, feel free to contact us at <support@sevensense.ch> with
your request.

- [I would like to mount Alphasense Core on my robot, what should I consider?](#i-would-like-to-mount-alphasense-core-on-my-robot-what-should-i-consider)
- [Are all images on Alphasense Core recorded at the same time?](#are-all-images-on-alphasense-core-recorded-at-the-same-time)
- [I don’t want to use ROS to access the data, what should I do?](#i-dont-want-to-use-ros-to-access-the-data-what-should-i-do)
- [Why is my sensor dropping frames?](#why-is-my-sensor-dropping-frames)
- [Can I use the Alphasense Core driver on an ARM architecture?](#can-i-use-the-alphasense-core-driver-on-an-arm-architecture)

## I would like to mount Alphasense Core on my robot, what should I consider?

In general, this depends on the application, but there are some common
guidelines which should be considered:
- First of all, please consult the
  [datasheet](https://drive.google.com/file/d/1jtchd_72k5LA9qdVV072_Ejv8v45Vt52/view)
  to find all relevant specifications such as the position of the mounting
  holes at the bottom, required power supply, or opening angles of the cameras.
- Related to that: Mount the sensor at a position where none of the cameras
  sees the hull of the robot. Note that the cameras have a wide field of view.
- If you plan to fuse the wheel odometry with your state estimation from the
  IMU/camera data, it is important that the there is a non-moving connection
  between the Alphasense Core mounting position and the wheel odometry frame.
- Depending on the kind of robot on which you are mounting Alphasense, we
  suggest to dampen the sensor in case you expect high-frequency vibrations,
  which can be relevant for a better state estimation output. This can for
  example be done with Silicone rubber dampers which are put between the sensor
  frame and the robot.
- The sensor is well suited for outdoor scenarios, especially the auto exposure
  is designed to work well despite the impact of sunlight. However, it has to
  be protected against rain/dust.

## Are all images on Alphasense Core recorded at the same time?

Yes, all image and IMU streams are hardware-synchronized. During auto exposure
operation, different cameras might have different exposure times, and they are
synchronized in the middle of the exposure time. The timestamp of an image
corresponds then to this mid-frame time.

## I don’t want to use ROS to access the data, what should I do?

We are currently working on a ROS-free API, which we will publish soon. Using
this, it will then be possible to use the image/IMU data directly in your C++
application independently of ROS or the catkin build system.

## Why is my sensor dropping frames?

Please check the following:
- Are you using a Gigabit Ethernet link? The driver will print a warning if
  this is not the case. Alternatively, this can also be verified manually by
  installing the `ethtool` package (`sudo apt install ethtool`) and checking
  that the output of `ethtool ETHERNET_INTERFACE_NAME | grep Speed` is
  `Speed: 1000Mb/s`. You can find the `ETHERNET_INTERFACE_NAME` with the
  following command:
  `find /sys/class/net -type l -not -lname '*virtual*' -printf '%f\n'`. A usual
  interface name is for example `eth0`.
- Are you using an USB/Ethernet adapter? Note that there can be issues with
  some adapters. If available, we suggest to use the built-in Ethernet port
  instead of a USB/Ethernet adapter.
- Does the configured frame rate fit the Gigabit Ethernet bandwidth? If this is
  not the case, try lowering the frame rate as described
  [here](/pages/sensor_settings.md#frame-rate-and-imu-frequency).

## Can I use the Alphasense Core driver on an [ARM architecture](https://en.wikipedia.org/wiki/ARM_architecture)?

Precompiled drivers for ARM will be released soon, please contact us if this is
relevant for your application.
