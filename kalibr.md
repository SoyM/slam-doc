# Kalibr

## kalibr_allan

* rosbag record /imu0 -O imu.bag

* rosrun bagconvert bagconvert imu.bag /imu0

* Run the included matlab scripts to generate an allan 

    deviation plot for the readings
> SCRIPT_allan_matparallel.m

* Interpret the generated charts to find noise values

> SCRIPT_process_result.m

## Collect calibration data

* rosrun topic_tools throttle messages <in_topic> 4.0 [out_topic]

* rosrun image_view image_view image:=/cam0/image_raw

* rosbag record /cam0/image_raw /cam1/image_raw -O static.bag

* rosrun image_view image_view image:=/cam0/image_raw

* rosbag record /cam0/image_raw /cam1/image_raw /imu0 -O dynamic.bag

## Run the calibration

动态标定的chain.yml 是 静态标定 产生的
动态标定需加 --time-calibration

* kalibr_calibrate_cameras --models pinhole-equi pinhole-equi --topics /cam0/image_raw /cam1/image_raw --bag static.bag --target aprilgrid_6x6.yaml

* kalibr_calibrate_imu_camera --cam chain.yaml --target aprilgrid_6x6.yaml --imu imu0.yaml --bag dynamic.bag --time-calibration

https://github.com/ethz-asl/maplab/wiki/Initial-sensor-calibration-with-Kalibr

## Get config

There are many tools to generate config for popular slam algorithm.

https://github.com/ethz-asl/kalibr/tree/master/aslam_offline_calibration/kalibr/python/exporters

example:

`rosrun kalibr kalibr_maplab_config --to-ncamera --label <your_label> --cam <your_cam_chain_yaml>`
