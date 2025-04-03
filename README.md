## XFeatVINS

目前在 Orin NX 上测试

环境推荐：
- JetPack 5.1.3
- Ubuntu 20.04
- OpenCV 4.X with CUDA
- ROS1 Noetic
- PyTorch 2.1.0
- CUDA 11.8

## 编译
```bash
git clone https://github.com/Derkai52/XFeatVINS.git
catkin_make

```

## 运行
```bash
roslaunch vins xfeatvins_rviz.launch config_file:=/home/emnavi/ws_fisheye_vins/src/VINS-Fisheye/config/fisheye_ptgrey_n3/fisheye_cuda.yaml

rosbag play fisheye_vins_2020-01-30-10-38-14.bag
```
