# Calibration

All Core Research Development Kits come with factory intrinsic and extrinsic calibration. We provide this calibration on our servers in several file formats.
The driver comes with a command to easily obtain the calibration files belonging to your sensor.

## Calibration model


### Camera intrinsics

The individual cameras are calibrated using a pinhole projection model. The distortion of the lens
is modelled with an equidistant model with five terms. See the reference below for more information
on the equidistant model.

> :information_source: **Info**: The pinhole+equidistant model is not the same 
  as the `plumb_bob` model that is commonly used in ROS. Because of our 
  wide-angle lenses the equidistant model is better suited.

*J. Kannala and S. Brandt (2006). A Generic Camera Model and Calibration Method for Conventional, Wide-Angle, and Fish-Eye Lenses, IEEE Transactions on Pattern Analysis and Machine Intelligence, vol. 28, no. 8, pp. 1335-1340* [(pdf link)](http://www.ee.oulu.fi/~jkannala/publications/tpami2006.pdf)

### Extrinsics

For each camera we provide the 6-DOF transformation with respect to the IMU.

## File formats

At the moment we produce two file formats `7s_sensors.yaml` and `kalibr_calib.yaml`. The main calibration file is `7s_sensors.yaml`. The others are generated to make our demos easy to use.

### 7s_sensors.yaml

All calibrated parameters are available in this file. An example can be found [here](/files/example_7s_sensors_dont_use.yaml).

For each camera there is a yaml node under `ncameras/cameras`, like the one below.

```yaml
- T_B_C: 
    rows: 4
    cols: 4
    data: [
    -0.0089889092, 0.0097254569, 0.9999123037, 0.0520086092,
    -0.9999261367, -0.0082678741, -0.0089086176, 0.0488993112,
    0.0081805087, -0.9999185256, 0.0097990577, -0.011757515,
    0.0, 0.0, 0.0, 1.0]
  camera:
    distortion:
      parameters: 
        rows: 4
        cols: 1
        data: [-0.0416026702, 0.0022689289, -0.0027567794, 0.000401603]
      type: equidistant
    id: 839e02677a1f4b5c91466b2bc5fdc33e
    image_height: 1080
    image_width: 1440
    intrinsics: 
      rows: 4
      cols: 1
      data: [701.4165958679, 701.480279171, 668.2392112416, 517.9783218077]
    label: /alphasense_driver_ros/cam0
    line-delay-nanoseconds: 0
    type: pinhole
```

The `label` node identifies to which camera the calibration belongs. In this
example the calibration is for camera 0.

```yaml
  label: /alphasense_driver_ros/cam0
```

The parameters for the intrinsic calibration are encoded in the `camera/intrinsics` and `camera/distortion` field for the projection model and distortion model respectively.

**Projection model**

The four parameters for the pinhole projection model are specified in the order `fu`, `fv`, `cu`, `cv`.
Where the pair `fu` `fv` is the focal length in u and v directions on the image plane, and the pair `cu` `cv` is the location of the center of projection in uv coordinates. The unit of all four parameters is pixels. The u and v directions run across the width and height of the image plane.

In this example `fu` = 701.4165958679, `fv`= 701.480279171, `cu` = 668.2392112416, and `cv` = 517.9783218077.

```yaml
- camera: 
    intrinsics: 
      data: [701.4165958679, 701.480279171, 668.2392112416, 517.9783218077]
```

**Distortion model**

The equidistant distortion model uses five parameters `k1`, `k2`, `k3`, `k4`, `k5` (see equation 6 of the reference mentioned above). We fix `k1` to 1, the other four parameters are encoded in `camera/distortion/parameters` in the same order.

In this example `k1` = 1, `k2`= -0.0416026702, `k3` = 0.0022689289, `k4` = -0.0027567794, and `k5` = 0.000401603.

```yaml
- camera:
    distortion:
      parameters: 
        data: [-0.0416026702, 0.0022689289, -0.0027567794, 0.000401603]
```

**Extrinsics**

Extrinsic calibration is encoded in the `T_B_C/data` node as a homogeneous transformation matrix with the translational part in meters. It expresses the pose of the camera in the frame of the IMU.

```yaml
- T_B_C: 
  data: [
    -0.0089889092,  0.0097254569,  0.9999123037,  0.0520086092,
    -0.9999261367, -0.0082678741, -0.0089086176,  0.0488993112,
     0.0081805087, -0.9999185256,  0.0097990577, -0.011757515,
     0.0,           0.0,           0.0,           1.0]
```



### kalibr_calib.yaml

This file is meant for the [stereo demo](https://github.com/sevensense-robotics/alphasense_stereo_demo) we provide on github.

## Obtaining the calibration files

The above mentioned files can be downloaded with the driver using the following
command.

```console
alphasense download_calibration - 7s_sensors.yaml
```

The command will output the file path on STDOUT. This can be used in a script
to automatically get the correct calibration even when switching the Alphasense
Core.

```bash
CALIBRATION_FILE_PATH=`alphasense download_calibration - 7s_sensors.yaml`
rosrun my_vio_package my_vio_node _calibration_file:=$CALIBRATION_FILE_PATH
```

The calibration can also be downloaded to a specific path on the computer

```console
alphasense download_calibration - 7s_sensors.yaml ~/my-favorite-path/my-calibration-file
```
