# calibrate

## imu

[IMU-Noise-Model](https://github.com/ethz-asl/kalibr/wiki/IMU-Noise-Model):

* Additive "White Noise"
* Bias

This has some nice utility scripts and packages that allow for calculation of the noise values for use in both kalibr and IMU filters.

https://github.com/rpng/kalibr_allan

## camera-imu

### camera 

* throttle is a ROS node that subscribes to a topic and republishes incoming data to another topic, either at a maximum bandwidth or maximum message rate.

rosrun topic_tools throttle messages <in_topic> 4.0 [out_topic]


https://github.com/ethz-asl/kalibr
