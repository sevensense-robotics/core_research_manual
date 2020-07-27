# Installation

The Alphasense Core driver binaries for Ubuntu are available from the Sevensense APT repositories.

* **Ubuntu 18.04/ROS Melodic**
  * AMD64 (Also known as x86_64) architecture: `deb [arch=amd64] https://deb.7sr.ch/alphasense bionic main`
  * ARM64 architecture: `deb [arch=arm64] https://deb.7sr.ch/alphasense bionic main`
* **Ubuntu 20.04/ROS Noetic**
  * Coming soon
  
In case you do not know the architecture of your system, you can execute:

```console
dpkg --print-architecture
```
  
## Adding the repository

First you need to trust the Sevensense key used to sign the packages.

```console
sudo apt install curl
curl -Ls https://deb.7sr.ch/pubkey.gpg | sudo apt-key add -  
```

Then you need to add the repository from the list above to your APT configuration.

```console
echo "deb [arch=$(dpkg --print-architecture)] https://deb.7sr.ch/alphasense bionic main" \
          | sudo tee /etc/apt/sources.list.d/sevensense.list
sudo apt update
```

## Installing the driver

The driver is split into several packages:

* `alphasense-driver-core`: Core driver package that always needs to be installed.
* `alphasense-viewer`: Viewer application to check the image streams.
* `alphasenese-firmware`: Package to distribute firmware upgrades.
* `ros-melodic-alphasense-driver-ros`: ROS interface to the driver.

To install **all packages** run:

```console
sudo apt install alphasense-driver-core alphasense-viewer alphasense-firmware ros-melodic-alphasense-driver-ros
```

To install the driver **without ROS** support:

```console
sudo apt install alphasense-driver-core alphasense-viewer alphasense-firmware
```

# Upgrading the firmware

The newest firmware can be flashed using the flashing tool included in the driver. Customers will receive a notification email when new firmware is available.

Follow these steps to upgrade to the newest firmware and driver:

> :warning: **Warning**: Do not unplug the power to the Alphasense Core while flashing is in progress. This can brick the device.

1. Upgrade the driver and firmware packages
   ```console
   sudo apt install alphasense-driver-core alphasense-viewer alphasense-firmware ros-melodic-alphasense-driver-ros
   ```
2. Connect **ONLY ONE** Alphasense Core to the host.
3. Run the upgrade utility, and confirm the upgrade.
   
   ```console
   flashalphasense
   ```
   
   ```console
   sevensense@7s-workstation:~$ flashalphasense
   Found device 'Sevensense AS1 720x540 (sn: 3A13199690341453 i: as1_ethernet l: 192.168.77.77)'
   Loading firmware from '/usr/lib/alphasense_firmware/data/fw_ethernet_206430361_IMX287_35.mcs.asf'.
   
   !IMPORTANT! Make sure the device has constant power while flashing is in progress. Interrupting power during flashing can break the device.
   Device to be flashed:   Sevensense AS1 720x540 (sn: 3A13199690341453 i: as1_ethernet l: 192.168.77.77)
   Firmware to be flashed: Alphasense firmware version 206430361, built on 2020-06-24T01:30:47 by Sevensense Robotics AG (206430361)
   Proceed with flashing [y/N]?
   y
   Start of flashing process, DO NOT TURN OFF DEVICE!
   Uploading firmware...
   100%            
   Writing to persistent memory...
   Flashing succeeded, please reboot the device!
   ```
4. When "Flashing succeeded, please reboot the device!" appears the Alphasense 
   Core can be rebooted by unplugging and then replugging the power.
