# Sensor settings

## Running the sensor with a settings file

Instead of starting Alphasense with the standard configuration, the driver can
be given a custom settings file. An example is provided here:
[ncamera_settings.yaml](/files/ncamera_settings.yaml). This can be passed to the
driver with one of the commands shown below.

Standalone viewer:

```console
viewalphasense --ncamera-settings /path/to/ncamera_settings.yaml
```

ROS driver:

```console
rosrun alphasense_driver_ros alphasense_driver_ros _ncamera_settings:=/path/to/ncamera_settings.yaml
```

## Dynamically configuring settings

When using the ROS driver, it is possible to change the settings dynamically as
an alternative to providing a settings file.
To adjust the configuration of the driver, use
[rqt_reconfigure](http://wiki.ros.org/rqt_reconfigure) after launching the
driver.

```
rosrun rqt_reconfigure rqt_reconfigure
```

This application lists all availabe settings and allows to change them on the
fly. Those are described in more detail in the sections below.

## Individual camera parameters

Each camera can have distinct settings for (auto) exposure and gain. The
exposure intervals of all cameras will be temporally aligned in the center.

### Camera exposure time

#### Fixed exposure time

The exposure time of a camera can be fixed by setting the
`auto_exposure_enabled` parameter to `false` and setting the
`manual_exp_time_us` to the desired exposure time in microseconds.

#### Auto exposure

Alphasense is equipped with an intelligent auto exposure algorithm which is
specially designed for computer vision and state estimation applications. It
detects areas in the image that contain high gradients and adjusts the exposure
time accordingly. As a result, it is possible to recognize structure in the
environment under the influence of sunlight, for example.

Auto exposure can be enabled by setting the `auto_exposure_enabled` parameter
to `true`. The algorithm adjusts the exposure time until it reaches the overall
image brightness defined in the `autoexp_target_median` setting. The target
brightness is encoded using 8-bit grayscale: 0 representing black and 255
white. The minimum and maximum exposure time range within which the algorithm
can operate can be configured with `min_exp_time_us` and `max_exp_time_us`
respectively. Both parameters are given in microseconds. If autogain is
enabled, the gain will be increased if the maximum exposure time is too low in
order to reach the desired image brightness. The parameter `autoexp_speed`
adjusts how smooth the change in exposure time and consequently image
brigthness is. However, when `autoexp_speed` is reduced, convergence is slowed
down.

##### Setting the Region Of Interest (ROI)

The auto exposure algorithm can be configured to only optimize brightness of a
certain rectangular patch of the image.

The boundaries of the patch can be set as follows:

```yaml
  autoexp_start_row: 0
  autoexp_end_row: -1
  autoexp_start_col: 0
  autoexp_end_col: -1
```

For `autoexp_end_row` and `autoexp_end_col`, -1 equals the maximum height and
width. So in this case, the whole image is being used.


### Camera gain

The camera gain parameter can be used to adjust light sensitivity of the
sensor. Increasing the gain will lead to brighter images with shorter exposure
times. Note that this comes at the cost of increased noise in the image.
Setting `manual_gain` to 0 will result in no amplification and 255 will
increase sensitivity by a factor of 5.6. If auto exposure is enabled, this
parameter has no influence.

### Auto gain

This setting is only relevant if auto exposure is enabled. Enabling this
parameter allows the auto exposure algorithm to increase the camera gain when
maximum exposure time is reached. This enables operation in very dark
environments or when maximum exposure time is too low to reach a sufficiently
bright image. In case the desired image brightness can be achieved adjusting
exposure time only, gain is kept at the lowest level to reduce noise. The
`autoexp_max_gain` allows for manual control over the maximum gain value, the
algorithm can apply.

### Rotating the image by 180 degrees

> :warning: **Breaks factory calibration**: Do not rotate the camera images when using the factory supplied calibration!

The camera image can be rotated by 180 degrees. This can be done by setting the
`rotate_image` parameter to `true`. The rotation of the images happens
directly on the Alphasense camera main board.

## Sensor parameters

### Number of enabled cameras

The number of enabled cameras can be set with the `num_cams` option. The camera
ports until the `num_cams` value will be enabled. By disabling camera ports, the
maximum allowed frame rate of the enabled cameras can be increased.

### Frame rate and IMU frequency

The camera frame rate can be set with the `camera_frequency_hz` option. The IMU
frequency can be set to 100, 200 or 400 Hz with the `imu_frequency_hz` option.

The minimum frame rate is 1Hz. The maximum frame rate depends on the number of cameras,
the maximum/manual exposure time, and network tuning parameters 
(`inter_packet_delay_us`/`pixels_per_packet`). The table below gives the absolute 
maximum frame rates for each number of cameras with exposure at 10us, `inter_packet_delay_us` at 1.0 us and `pixels_per_packet` at 7200.

#### Absolute maximum frame rates in Hz/fps

| Number of Cameras  | Alphasense Core (0.4 MP, Mono) | Alphasense Core (1.6 MP, Mono) |
| --- | --- | ---|
| 1 | 75 | 30 |
| 2 | 75 | 30 |
| 3 | 75 | 24 |
| 4 | 72 | 18 |
| 5 | 58 | 14 |
| 6 | 48 | 12 |
| 7 | 41 | 10 |
| 8 | 35 | 8 |

### Gyro sensitivity

The sensitivity of the gyroscope can be set with the `gyro_range_deg_per_sec`.
The parameter sets the maximum measurable rotation rate. Setting this value
higher will decrease the gyro sensitivity.

### Network tuning

The peak bandwidth used by the Alphasense Core can be set with the `peak_bandwidth_limit_mbps` parameter.

The Alphasense Core packet size can be set with the `pixels_per_packet` option.
It is best to increase the `pixels_per_packet` to the highest possible allowed by your network setup (The maximum is 7200).

> :information_source: **Info**: The MTU of the network card to which the Alphasense Core is connected needs to be at least `pixels_per_packet` + 60, otherwise the driver cannot receive the image stream packets. In that case you will get "Image receive timed out." errors.

See [Maximize network performance](/pages/maximize_network_performance.md) for more information. 
