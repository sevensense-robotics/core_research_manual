# Getting started with Alphasense Core

Congratulations on your Alphasense Core 5-Camera Development Kit!  Start using
it by following these simple steps:

## What's in the box?

- **The sensor**: a development kit with 5 cameras and an IMU (synchronized)
- **Ethernet cable**: for transferring the data
- **AC adapter to sensor cable**: for powering the sensor
- **12V 25W AC wall adapter** (230V Euro plug)

## Installing the driver

Ubuntu 18.04 and a [ROS Melodic
installation](http://wiki.ros.org/melodic/Installation/Ubuntu) are required to
install the driver.  A `ROS-Base` installation is sufficient, but we recommend
the `Desktop Install` to use the `rqt` GUI tools mentioned in other parts of
this manual.

The following instructions describe the steps for adding the Sevensense APT
repository and installing the Alphasense driver contained therein.

```
# Install curl.
sudo apt install curl

# Add the Sevensense PGP key to make this machine trust Sevensense's packages.
curl -Ls https://deb.7sr.ch/pubkey.gpg | sudo apt-key add -

# Add the Sevensense APT repository to the list of known sources.
echo "deb [arch=amd64] https://deb.7sr.ch/alphasense bionic main" \
          | sudo tee /etc/apt/sources.list.d/sevensense.list

# Install the Alphasense driver.
sudo apt update
sudo apt install ros-melodic-alphasense-driver-ros
```

## Setting up the network configuration

Open a terminal, launch the `nm-connection-editor` and add a new `Ethernet` connection:

![nm_connection_editor](/images/nm_connection_editor.png)

Name the new connection `alphasense`. Under the `IPv4 Settings` tab, change the
method to `Manual` and add the following static IP address:

![ip_settings](/images/alphasense_ip_setting.png)

Save the new configuration.

## Running the driver

Connect the sensor to the power cable and the Ethernet cable.  Click on the
Network Manager icon (usually located in the top bar of Ubuntu) and select the
`alphasense` network connection.

There are two ways to run the Alphasense driver, described in the following two
sections:

### 1) Launching the standalone viewer

Launch the standalone viewer with the following command:

```
viewalphasense
```

When running the above command for the first time, the socket buffer size may
have to be increased.  The error message will contain the corresponding command
to do this:

```
sudo sysctl -w net.core.rmem_max=3145728
```


As an alternative to the command above, the buffer size can be increased
permanently by creating a file called
`/etc/sysctl.d/90-increase-network-buffers.conf` with the following content:

```
net.core.rmem_max=3145728
```

After that, you can launch the Alphasense GUI which will display all available
image streams:

![viewalphasense](/images/viewer.png)

### 2) Launching the ROS driver

As an alternative to using the standalone viewer and in order to access the
streamed data, our ROS driver can be used.  Note that only one driver at a time
can be running, so `viewalphasense` needs to be stopped before launching this
one.  To start the ROS driver of Alphasense, run the following command:

```
roscore&
rosrun alphasense_driver_ros alphasense_driver_ros
```

Via ROS messages we offer an easy way to access the images and the IMU data.
You can then view all available ROS topics using the following command:

```
rostopic list
```

We recommend tools such as [rqt_image_view](http://wiki.ros.org/rqt_image_view)
and [rqt_plot](http://wiki.ros.org/rqt_plot) to inspect the images and the IMU
data.

Find out more about the sensor settings and how to permantently set them
[here](/pages/sensor_settings.md).
