# Frequently asked questions

We would like to answer frequently asked questions on this page. If you do not
find an answer here, feel free to contact us at <support@sevensense.ch> with
your request.

- [I would like to mount Core Research on my robot, what should I consider?](#i-would-like-to-mount-alphasense-core-on-my-robot-what-should-i-consider)
- [Are all images on Core Research recorded at the same time?](#are-all-images-on-alphasense-core-recorded-at-the-same-time)
- [I don’t want to use ROS to access the data, what should I do?](#i-dont-want-to-use-ros-to-access-the-data-what-should-i-do)
- [Why is my sensor dropping frames?](#why-is-my-sensor-dropping-frames)
- [Can I use the Core Research driver on an ARM architecture?](#can-i-use-the-alphasense-core-driver-on-an-arm-architecture)
- [How do I change the IP address of the sensor?](#how-do-i-change-the-ip-address-of-the-sensor)
- [Can I synchronize the internal clock of the Core Research to an external system?](#can-i-synchronize-the-internal-clock-of-the-alphasense-core-to-an-external-system)
- [How do I fix "Image receive timed out"?](#how-do-i-fix-image-receive-timed-out)
- [Why is my Core Research not detected?](#why-is-my-alphasense-core-not-detected)
- [How do I find the serial number of my Core Research?](#how-do-i-find-the-serial-number-of-my-alphasense-core)


## I would like to mount Core Research on my robot, what should I consider?

In general, this depends on the application, but there are some common
guidelines which should be considered:
- First of all, please consult the
  [datasheet](https://hubs.ly/Q01XVycm0)
  to find all relevant specifications such as the position of the mounting
  holes at the bottom, required power supply, or opening angles of the cameras.
- Related to that: Mount the sensor at a position where none of the cameras
  sees the hull of the robot. Note that the cameras have a wide field of view.
- If you plan to fuse the wheel odometry with your state estimation from the
  IMU/camera data, it is important that the there is a non-moving connection
  between the Core Research mounting position and the wheel odometry frame.
- Depending on the kind of robot on which you are mounting Alphasense, we
  suggest to dampen the sensor in case you expect high-frequency vibrations,
  which can be relevant for a better state estimation output. This can for
  example be done with Silicone rubber dampers which are put between the sensor
  frame and the robot.
- The sensor is well suited for outdoor scenarios, especially the auto exposure
  is designed to work well despite the impact of sunlight. However, it has to
  be protected against rain/dust.

## Are all images on Core Research recorded at the same time?

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
- See the [Maximize network performance](/pages/maximize_network_performance.md) page for guidelines on configuring the system and Core Research for maximal performance.
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

## Can I use the Core Research driver on an [ARM architecture](https://en.wikipedia.org/wiki/ARM_architecture)?

Yes this is possible. You can install the driver for the `arm64` architecture, see [Installation](/pages/installation_and_upgrade.md#installation).

## How do I change the IP address of the sensor?

The IP can be changed by following the instructions in 
[Modifying persistent configuration](/pages/configuring_the_network.md#modifying-persistent-configuration).

## Can I synchronize the internal clock of the Core Research to an external system?

Yes, the Core Research supports the PTP (Precision Time Protocol) protocol to synchronize the internal clock over the
ethernet connection. Follow the instructions in [Synchronized time](/pages/time_synchronization.md#time-synchronization) to set this up.

## How do I fix "Image receive timed out"?

The "Image receive timed out" means that the driver is not receiving image stream packets from the Core Research. This can happen because of many reasons. Some commonly occuring reasons are:

* The Core Research is not powered anymore or the ethernet cable is unplugged.
* The network configuration has been changed.
* The Core Research is configured to send data to a different host IP, this can be checked with `alphasense show -`.
* There is another driver running. Only one driver can be running at a time.
* The `pixels_per_packet` setting is set higher than the MTU of the network interface. See [Network interface MTU](/pages/maximize_network_performance.md#network-interface-mtu).


## Why is my Core Research not detected?

When the Core Research is not detected with `alphasense list`, check the following points:

* The Core Research is powered, there should be a green LED blinking directly on the main PCB behind the ethernet connector.
* There is an active ethernet connection. There should be two LEDs on the ethernet port, one should be lit constantly, the other blinking.
* The network profile you created is activated. It is easiest to verify this by running `ip address` in the terminal and check if the interface has the correct IP address assigned.
* The firewall is disabled.
* No network related errors are printed in `dmesg`.

When the above points do not solve the problem. A conflicting network configuration is often the problem. Double check all custom network configuration.

To check that the Core Research is actually sending out packets `tcpdump` can be used (`INTERFACE_NAME` should be replaced with the name of the interface the Core Research is connected to):

```console
sevensense@7s-workstation:~$ sudo tcpdump -i INTERFACE_NAME udp port 5349 -vv -nn -n
tcpdump: listening on enp5s0, link-type EN10MB (Ethernet), capture size 262144 bytes
11:22:56.792794 IP (tos 0x0, ttl 64, id 0, offset 0, flags [DF], proto UDP (17), length 221)
    192.168.77.77.5349 > 255.255.255.255.5349: [no cksum] UDP, length 193
11:22:56.912826 IP (tos 0x0, ttl 64, id 0, offset 0, flags [DF], proto UDP (17), length 221)
    192.168.77.77.5349 > 255.255.255.255.5349: [no cksum] UDP, length 193
11:22:57.032827 IP (tos 0x0, ttl 64, id 0, offset 0, flags [DF], proto UDP (17), length 221)
    192.168.77.77.5349 > 255.255.255.255.5349: [no cksum] UDP, length 193
```

## How do I find the serial number of my Core Research?

The serial number of the Core Research can be found by connecting it to a computer and running `alphasense show -`

The serial number in the example below is `3A1319969034225F`.

```
sevensense@7s-workstation:~$ alphasense show -

...

Device information:
 baseboard_imu_type          BMI085
 cam0_id                     not-connected
 cam1_id                     not-connected
 cam2_id                     not-connected
 cam3_id                     not-connected
 cam4_id                     not-connected
 cam5_id                     not-connected
 cam6_id                     not-connected
 cam7_id                     not-connected
 firmware_version            125389894
 serial_number               3A1319969034225F
 fpga_size                   35
 fpga_temperature_degrees_c  49
 image_sensor_type           IMX287
```
