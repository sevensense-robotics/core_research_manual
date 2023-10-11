# Getting started with Core Research

Congratulations on your Core Research 5-Camera Development Kit!  Start using
it by following these simple steps:

## What's in the box?

- **The sensor:** a development kit with 5 cameras and an IMU (synchronized)
- **Cable adapter** from the 2.1x5.5 mm Barrel Plug of the AC adapter to the Molex Nano-Fit connector of the sensor

## What else is needed?

- A **Category 6 (Cat6) Ethernet cable** to transfer the data to your device
- A **12V 25W AC wall adapter** with a 2.1x5.5 mm Barrel Plug that mates to the adapter cable (eg. [CUI SWI25-12-E-P5 with an EU plug](https://www.mouser.ch/ProductDetail/490-SWI25-12-E-P5))

## Installing the driver

> :information_source: **Info**: Have a look at the
  [Installation](/pages/installation_and_upgrade.md) page for a list of supported Ubuntu versions and processor architectures.

Ubuntu 18.04 or 20.04 is required to install the driver. We also recommend installing ROS Melodic/Noetic `desktop-full` ([ROS 
installation](http://wiki.ros.org/melodic/Installation/Ubuntu)) A `ros-base` installation is sufficient, but we recommend
the `desktop-full` to use the `rqt` GUI tools mentioned in other parts of
this manual.

The following instructions describe the steps for adding the Sevensense APT
repository and installing the Alphasense driver contained therein.

```
# Install curl.
sudo apt install curl

# Add the Sevensense PGP key to make this machine trust Sevensense's packages.
curl -Ls http://deb.7sr.ch/pubkey.gpg | sudo gpg --dearmor -o /usr/share/keyrings/deb-7sr-ch-keyring.gpg

# Add the Sevensense APT repository to the list of known sources.
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/deb-7sr-ch-keyring.gpg] http://deb.7sr.ch/alphasense/stable $(lsb_release -cs) main" \
          | sudo tee /etc/apt/sources.list.d/sevensense.list

# Install the Alphasense driver.
sudo apt update
sudo apt install alphasense-driver-core alphasense-viewer alphasense-firmware ros-melodic-alphasense-driver-ros ros-melodic-alphasense-driver
```

## Setting up the network configuration

In order to connect your Core Research device to your computer, a network
configuration with a static IP needs to be set up.

To set this up, open a terminal, launch the `nm-connection-editor` and add a
new `Ethernet` connection:

![nm_connection_editor](/images/nm_connection_editor.png)

Name the new connection `alphasense`. Under the `IPv4 Settings` tab, change the
method to `Manual` and add the following static IP address:

![ip_settings](/images/alphasense_ip_setting.png)

Save the new configuration.

By default the sensor is configured with the following network settings.

| Setting  | Value |
| ------------------- | ------------- |
| Host IP Address     | 192.168.77.78  |
| Host Netmask        | 255.255.255.0  |
| Sensor IP Address   | 192.168.77.77  |

## Running the driver

Connect the sensor to the power cable and the Ethernet cable.  Click on the
Network Manager icon (usually located in the top bar of Ubuntu) and select the
`alphasense` network connection.

There are two ways to run the Alphasense driver, described in the following two
sections:

### 1) Launching the standalone viewer

First increase the maximum allowed socket buffer size:

```
sudo sysctl -w net.core.rmem_max=11145728
```

This setting can be made permanent by creating a file called
`/etc/sysctl.d/90-increase-network-buffers.conf` with the option:

```console
echo -e "net.core.rmem_max=11145728" | sudo tee /etc/sysctl.d/90-increase-network-buffers.conf
```

After that, you can launch the Alphasense GUI which will display all available
image streams.

Launch the Alphasense GUI with the following command:

```
viewalphasense
```

![viewalphasense](/images/viewer.png)

```console
sevensense@7s-workstation:~$ viewalphasense
Found camera, opening...
[ INFO|2020-07-21 15:53:21.389801] Assuming camera is connected to NIC enp5s0.
Opened! writing settings...
[ INFO|2020-07-21 15:53:21.391719] Calculated an inter packet delay of 1 us.
[ INFO|2020-07-21 15:53:21.593148] Reallocating image stream packet buffers (6990 packets @ 1500 bytes).
[ INFO|2020-07-21 15:53:21.595462] Resizing image socket buffer to 10485760 bytes.
Ready!
202 IMU measurements received in 1 second.
|- Current IMU measurement timestamp: 12 190998900
|- accl.x: -1.6711
|- accl.y: -0.0843384
|- accl.z: -9.47461
|- gyro.x: 0.000213059
|- gyro.y: 0.0288695
|- gyro.z: -0.00255671
```

### 2) Launching the ROS driver

As an alternative to using the standalone viewer and in order to access the
streamed data, our ROS driver can be used.  Note that only one driver at a time
can be running, so `viewalphasense` needs to be stopped before launching this
one.  


If not already done before, increase the maximum allowed socket buffer size:

```
sudo sysctl -w net.core.rmem_max=11145728
```

Start the ROS driver of Alphasense, run the following command:


```
source /opt/ros/melodic/setup.bash
roscore&
rosrun alphasense_driver_ros alphasense_driver_ros

```

Via ROS messages we offer an easy way to access the images and the IMU data. 
See [ROS Interface](/pages/ros_driver_usage.md) for more information on the provided 
topics and configuration parameters.

We recommend tools such as [rqt_image_view](http://wiki.ros.org/rqt_image_view)
and [rqt_plot](http://wiki.ros.org/rqt_plot) to inspect the images and the IMU
data.

Find out more about the sensor settings and how to permanently set them
[here](/pages/sensor_settings.md).
