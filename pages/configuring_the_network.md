# Configuring the network

The network settings of the device can be configured using the command line tools that are supplied with the driver.

## Setting up the host computer

The host computer (the computer you connect the sensor to) needs be assigned 
a static IP address on the same subnet as the Alphasense Core. The steps for 
setting this up depends on how the networkis managed on the host. We provide 
steps for a standard desktop Ubuntu setup and a standard server Ubuntu setup. 

**Default Alphasense Core network settings**

| Setting  | Value |
| ------------------- | ------------- |
| Host IP Address     | 192.168.77.78  |
| Host Netmask        | 255.255.255.0  |
| Sensor IP Address   | 192.168.77.77  |

### Setting up a static IP on Ubuntu desktop

Follow the steps in the [Getting started with Alphasense Core](/pages/getting_started.md#setting-up-the-network-configuration) to configure
the network through the Network Manager GUI in Ubuntu.

### Setting up a static IP on Ubuntu server

> :warning: **Warning**: The instructions below might not work properly when the network is 
>being managed by a network manager like "NetworkManager" on Ubuntu desktop. In that case you 
>have to consult the manual of the network manager on how to assign a static IP.

First you need to figure out to which network interface the Alphasense Core is connected. A list of network interfaces can be obtained with

```console
ip link
```

```console
sevensense@7s-workstation:~$ ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: enp5s0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq state DOWN mode DEFAULT group default qlen 1000
    link/ether 68:05:ca:b1:2a:76 brd ff:ff:ff:ff:ff:ff
3: eno1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether 04:92:26:5d:04:9c brd ff:ff:ff:ff:ff:ff
4: enp3s0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 68:05:ca:b8:09:8a brd ff:ff:ff:ff:ff:ff
5: enx000acd338ed4: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc fq_codel state DOWN mode DEFAULT group default qlen 1000
    link/ether 00:0a:cd:33:8e:d4 brd ff:ff:ff:ff:ff:ff
```

When the network interface is known it needs to be set up and assigned a static IP.

```console
sudo ip link set INTERFACE_NAME up
sudo ip address add dev INTERFACE_NAME 192.168.77.78/24
```

For example to do this for the interface `enp3s0` execute

```console
sevensense@7s-workstation:~$ sudo ip link set enp3s0 up
sevensense@7s-workstation:~$ sudo ip address add dev enp3s0 192.168.77.78/24
```

The assingment is not permanent and will be lost after reboot. 
We recommend looking at https://netplan.io/ for setting up a permanent network configuration.

## Listing devices attached to the network

Once the network is setup, a list of all Alphasense Core sensors attached to the network can be obtained with `alphasense list`.

```
sevensense@7s-workstation:~$ alphasense list
- Sevensense AS1 720x540 (sn: 3A1319969034225F i: as1-ethernet l: 192.168.1.128)
```

The Alphasense Core sensors are identified by their serial number, 
which for the sensor in the example above is *3A1319969034225F*.

If your Alphasense Core does not show up in the list, the following should be checked:

1. Is the Alphasense Core powered on, and the ethernet cable connected?
2. Is the LED on the sensor's ethernet port blinking? If not check the ethernet cable.
3. Is your network configured correctly? In case of a new sensor check out the
[Setting up the network configuration](/pages/getting_started.md#setting-up-the-network-configuration) 
section. You might have to reactivate the network profile after reconnecting the ethernet cable. You can check if the network interface is up and has an IP assigned by exeucting `ip address`
4. When you are not sure of the network configuration anymore, 
see [Recovering a device with unknown network configuration](/pages/configuring_the_network.md#recovering-a-device-with-an-unknown-network-configuration).

> :information_source: **Info**: The Alphasense Core does not respond to pings. This does not mean the device is broken.

## Showing device information

More information about a device can be viewed using the `alphasense show SERIAL` command. If only one device is attached
the serial number can be replaced with `-` to get information about the first found device.

> :warning: **Warning**: Do not run this command with the driver running. This command might interrupt the image and IMU streams.

```
sevensense@7s-workstation:~$ alphasense show 3A1319969034225F
Device: Sevensense AS1 720x540 (sn: 3A1319969034225F i: as1-ethernet l: 192.168.1.128)

Device persistent configuration:
 device_ip                   192.168.77.77
 gateway_ip                  192.168.77.1
 host_ip                     192.168.77.78
 mac_address                 70:b3:d5:9c:30:36
 subnet_mask                 0.0.0.0

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

## Modifying persistent configuration

Persistent configuration options like `device_ip` can be modified using the `alphasense set` command. 
To for example change the sensors IP to `192.168.77.76` and the subnet mask to `255.255.0.0`, run the command below. 

```
alphasense set 3A1319969034225F device_ip 192.168.77.76 subnet_mask 255.255.0.0
```

When the sensors IP is changed an exception might be shown, this is because the
sensor is not reachable anymore under the old IP.

```
sevensense@7s-workstation:~$ alphasense set 3A1319969034225F device_ip 10.0.0.1
terminate called after throwing an instance of 'alphasense::AS1NetworkException'
  what():  exchangeSettingsWithFpga maximum number of retries reached.
```

The Alphasense Core sends the image streams to a preconfigured host IP.
The factory default host IP is `192.168.77.78`. The command below can be used to change
this host IP. In the example the host IP is changed to `192.168.77.177`

```
alphasense set 3A1319969034225F host_ip 192.168.77.177
```

## Recovering a device with an unknown network configuration

When the devices network settings are modified and the sensor is outside of the subnet configured on 
the host computer. It will not be discoverable anymore by the driver. This is because of the reverse 
path filtering in the linux kernel, which blocks broadcasts sent from an IP not within the subnet.

```
sevensense@7s-workstation:~$ alphasense list
No devices found.
```

Disable reverse path filtering by executing the following two commands, replacing `INTERFACE_NAME` with 
the name of the interface the device is connected to (for example `eth0` or `enp1s0`):

```
sudo sysctl net.ipv4.conf.all.rp_filter=0
sudo sysctl net.ipv4.conf.INTERFACE_NAME.rp_filter=0
```

With reverse path filtering disabled the device can be discovered (the discovered IP address 
is `10.1.1.77`).

```
sevensense@7s-workstation:~$ alphasense list
- Sevensense AS1 720x540 (sn: 3A1319969034324F i: as1_ethernet l: 10.1.1.77)
```

To be able to communicate with the device the host computer's network has to be configured with a 
subnet that contains the IP of the device (`10.1.1.77`). It is recommended to set the host computer's 
IP to either one IP higher or lower than the device in this case `10.1.1.76` or `10.1.1.78`. This new IP has to be set as host IP for the Alphasense Core using:

```
alphasense set 3A1319969034324F host_ip 10.1.1.76
```

The device
should now be accessible again. 

If you want to reset the factory default network settings, run the following command:

```
alphasense set 3A1319969034324F device_ip 192.168.77.77 host_ip 192.168.77.78 subnet_mask 0.0.0.0 gateway_ip 192.168.77.1
```

After changing the network settings the device can be discovered again with the default network settings.

```
sevensense@7s-workstation:~$ alphasense list
- Sevensense AS1 720x540 (sn: 3A1319969034324F i: as1_ethernet l: 192.168.77.77)
```

Reverse path filtering can now be enabled again:

```
sudo sysctl net.ipv4.conf.all.rp_filter=1
sudo sysctl net.ipv4.conf.INTERFACE_NAME.rp_filter=1
```