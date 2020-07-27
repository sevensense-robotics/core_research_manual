# Maximize network performance

To maximize the performance of the Alphasense Core some default configuration
options need to be changed. Network packet drop and therefore frame drops 
can be eliminated by following the guidelines below.

The guidelines are split into the following three categories:

1. Hardware setup
2. Kernel and network configuration
3. Alphasense Core configuration

## Hardware setup

We recommend using a modern PCIe gigabit ethernet card that supports jumbo frames 
(for example cards based on the Intel I2xx controllers).
USB to Ethernet adapters are not recommended for a final setup. 
They will work but aren't as reliable and might drop frames.

Some embedded platforms like for example the NVidia Jetson TX2 cannot reliably support
the full gigabit bandwidth on their integrated ethernet ports and will have a reduced maximum frame rate.

If the Alphasense Core is connected over a switch, make sure that it supports 
gigabit ethernet speeds. When multiple devices are connected to the switch and the 
Alphasense Core is sharing bandwidth with other sensors, a part of the bandwidth 
will need to be reserved for the Alphasense Core. The achievable maximum frame rate will 
depend on the amount of bandwidth reserved for the Alphasense Core. Note that this reserved
bandwidth needs to be configured as the peak bandwidth limit, 
see [Alphasense Core configuration](/pages/maximize_network_performance.md#alphasense-core-configuration).

The graphs below show the maximum achievable frame rate for a give number of sensors 
and amount of reserved bandwidth.

![nm_connection_editor](/images/alphasense_core_04mp_bandwidth.png)

![nm_connection_editor](/images/alphasense_core_16mp_bandwidth.png)

Correct cabling that supports gigabit ethernet speeds must be used in the network 
between the host and the Alphasense Core. In most cases this means CAT5e or CAT6 cables.

## Kernel and network configuration

### Network stack parameters

The Linux kernel has a lot of parameters that can be tweaked for better network performance. We recommend
 doubling some of the default settings. 

```console
sudo sysctl -w net.core.netdev_budget=600
sudo sysctl -w net.core.netdev_max_backlog=2000
```

These changes will reset after reboot. They can be made persistent by creating the following file:

```console
echo -e "net.core.netdev_budget=600\nnet.core.netdev_max_backlog=2000" | sudo tee /etc/sysctl.d/90-alphasense-network-parameters.conf
```

### Network interface ring buffer

We also recommend increasing the ring buffer of the network card. 
First check the maximum supported RX ring size by executing:

```console
sudo apt install ethtool
sudo ethtool -g INTERFACE_NAME
```

```console
sevensense@7s-workstation:~$ sudo ethtool -g eth0
Ring parameters for eth0:
Pre-set maximums:
RX:		4096
RX Mini:	0
RX Jumbo:	0
TX:		4096
Current hardware settings:
RX:		256
RX Mini:	0
RX Jumbo:	0
TX:		256
```

The maximum RX ring size in the above case is 4096. The ring size can be increased using:

```console
sudo ethtool -G INTERFACE_NAME rx 4096
```

The change in ring size does not persist across reboots. Some network managers support configuration
of the ring sizes. Otherwise it can be added to a script that runs on boot.

### Network interface MTU

The last thing to configure is the Maximum Transmission Unit (MTU) of the network interface.
This needs to be increased to 7260 to allow for the maximum packet size supported by the Alphasense Core.

```console
sudo ip link set INTERFACE_NAME mtu 7260 
```

This setting does also not persist across reboots and needs to be configured in the 
network manager or a startup script.

## Alphasense Core configuration

Three driver configuration parameters influence the network performance of the Alphasense Core
`pixels_per_packet`, `image_socket_receive_buffer_size_mb`, and `peak_bandwidth_limit_mbps`.
See [Sensor settings](/pages/sensor_settings.md) for instructions on how to supply the driver
with these parameters.

`pixels_per_packet` configures the packet size used for the video stream. Bigger packets decrease the network stack overhead, because less packets are needed to transfer a frame. This should be set
to the maximum of 7200 when possible. This is limited by the maximum MTU of your network interface.
Note that the network card has to be configured to accept bigger packets, 
see [Network interface MTU](/pages/maximize_network_performance.md#network-interface-mtu)

`image_socket_receive_buffer_size_mb` sets the size of the buffer used for the video stream
socket. The default value for this is 10 megabyte which should be enough for most systems. This cannot be
set higher than the maximum socket buffer size allowed by the kernel. The maximum the kernel allows can
be changed with (the value is in bytes):

```console
sudo sysctl -w net.core.rmem_max=11145728
```

`peak_bandwidth_limit_mbps` sets the maximum peak bandwidth in megabits per second of the Alphasense Core.
This should be set to 1000, which means that it transfers the captured frames to the host at full gigabit speed.
For some systems this can be too much and cause buffer overflows in lower levels of the network stack. In that
case it can be tuned to a lower value at the cost of decreased maximum frame rate and higher transfer latency. This parameter also needs to be lowered when the Alphasense Core has to share the bandwidth
with other sensors or devices.
