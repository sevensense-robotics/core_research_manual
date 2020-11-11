# Time synchronization

The Alphasense Core stamps all measurements using its internal clock. The camera
frames are stamped at the middle of their exposure intervals. This means that the
IMU and cameras are all inside the same time frame. In case no other sensors are used
this can be treated as the global time frame and no further action has to be taken.
However, if measurements from other cameras, LiDARs, sonars or wheel odometery are 
present and are intended to be fused with the measurements of Alphasense Core, time 
synchronization between those sensors is necessary.

We distinguish three possible modes of setting up the Alphasense Core & driver:

* **Raw time:** In this mode the ROS messages will contain the raw timestamp of the internal
device clock. This clock starts counting from timestamp 0 on device power up and has a non-constant drifting offset with respect to ROS time.
* **Translated time:** (default) In this mode the raw timestamps of the internal clock are translated to the host time frame using the [cuckoo_time_translator](https://github.com/ethz-asl/cuckoo_time_translator). A constant offset with respect to ROS time remains. This offset could be obtained by a suitable calibration procedure (depends on the sensor setup and is not provided by Sevensense).
* **Synchronized time:** In this mode the internal device clock is synchronized to the host computer using the PTP protocol. This puts the device clock into the same time frame as the host and the raw timestamps can be directly used for sensor fusion.

The instructions for configuring each mode are given below.

## Raw time

Raw timestamps can be enabled by disabling the time translation functionality in the ROS driver. This can be done by setting the ROS parameter `translate_device_time` to `False` (see [ROS Parameters](/pages/ros_driver_usage.md#parameters)). 

Make sure that no PTP master is present in the network, because otherwise the internal clock would be adjusted to be 
in sync with the master.

## Translated time

This mode can be enabled by setting the ROS parameter `translate_device_time` to `True`. Note that `translate_device_time` is `True` by default.

## Synchronized time (PTP)

> :information_source: **Info**: In the current firmware/driver version (<= 1.7.1) PTP synchronization will cause an IMU timestamp warning to trigger "IMU measurement period unstable, error: 0.1000ms.". This can be ignored and will be fixed in the next firmware release.

The Alphasense Core will automatically start the synchronization when a PTP master is connected to the ethernet network. Only PTP over UDP/IPv4 is supported. See [Running a PTP master on linux](#running-a-ptp-master-on-linux) on how to run a PTP master directly on the host.

> :warning: **Important**: Time translation needs to be **disabled** when PTP is used.

Time translation needs to be disabled for the internal clock time to be passed through correctly. See [Raw time](#raw-time) above on how to disable this. Note that raw and synchronized time are essentially the same from the point of view of the driver. It's the presence of a PTP master in the network that makes the device change its internal clock.

### Running a PTP master on linux

In this example we use tools from the [linuxptp project](https://sourceforge.net/projects/linuxptp/). This example is just to quickly setup a functioning PTP master, please see the linuxptp documentation for more advanced command line flags.

The tools can be installed with:

```sudo apt install linuxptp```

In all commands below you need to replace `INTERFACE_NAME` with the name of the network interface to which the Alphasense Core is connected.

We recommend using a network interface that has hardware timestamping support. Most modern PCIe network interfaces support this, USB to Ethernet adapters usually do not support this. You can use the following command to check if your network card supports hardware timestamping:

`ethtool -T INTERFACE_NAME`

The console output should look similar as the one shown below:

```console
sevensense@7s-workstation:~$ sudo ethtool -T enp5s0
Time stamping parameters for enp5s0:
Capabilities:
	hardware-transmit     (SOF_TIMESTAMPING_TX_HARDWARE)
	software-transmit     (SOF_TIMESTAMPING_TX_SOFTWARE)
	hardware-receive      (SOF_TIMESTAMPING_RX_HARDWARE)
	software-receive      (SOF_TIMESTAMPING_RX_SOFTWARE)
	software-system-clock (SOF_TIMESTAMPING_SOFTWARE)
	hardware-raw-clock    (SOF_TIMESTAMPING_RAW_HARDWARE)
PTP Hardware Clock: 0
Hardware Transmit Timestamp Modes:
	off                   (HWTSTAMP_TX_OFF)
	on                    (HWTSTAMP_TX_ON)
Hardware Receive Filter Modes:
	none                  (HWTSTAMP_FILTER_NONE)
	all                   (HWTSTAMP_FILTER_ALL)

```

Hardware timestamping is possible if the capabilities `SOF_TIMESTAMPING_TX_HARDWARE`, `SOF_TIMESTAMPING_RX_HARDWARE`, and `SOF_TIMESTAMPING_RAW_HARDWARE` are available.

When hardware timestamping is used, the internal clock of the network card first needs to be synchronized with the system clock. This can be done by running:

`sudo phc2sys -m -c INTERFACE_NAME -s CLOCK_REALTIME -O 0 -u 10`

This command needs to be kept running in the background to make sure the clock does not drift away again. The command will print information about the synchronization status every 10 seconds. When software timestamping is used this command can be skipped.

The PTP master can be started with the command below (to use software timestamping replace `-H` with `-S`):

`sudo ptp4l -m -H -i INTERFACE_NAME`

This command also needs to be kept running in the background.
