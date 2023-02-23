# Helmet-YOLOv5 Deploy to RK3568

## Prerequisite
1、下载第三方库opencv：
链接：https://pan.baidu.com/s/1CvXOsnHHaZzxcMh_-x5Ffg 提取码：qkr7
将3rdparty_yolov5s_rknn_deploy/rknn_to_deploy_3rdparty/opencv放到rknn_to_deploy/examples/3rdparty/中
2、 安装RKNN-Toolkit 

1. 安装 Python3.6 和 pip3，也可以用conda创建一个虚拟环境
sudo apt-get install python3 python3-dev python3-pip
2. 安装相关依赖
sudo apt-get install libxslt1-dev zlib1g zlib1g-dev libglib2.0-0 libsm6 \
libgl1-mesa-glx libprotobuf-dev gcc
3、安装python相关环境
cd onnx_to_rknn
pip install -r requirements-1.1.0.txt
4、安装rknn-toolkit
进入到3rdparty_yolov5s_rknn_deploy/onnx_to_rknn_3rdparty第三方库中
sudo pip3 install rknn_toolkit2*.whl
5、检查 RKNN-Toolkit 是否安装成功
rk@rk:~/rknn-toolkit2/package$ python3
>>> from rknn.api import RKNN
>>>

## Helmet Yolov5 to ONNX

将yolov5s.pt转成yolov5s.onnx

```
cd yolov5s-to-onnx
```

1、安装依赖环境，跟上述环境有重叠的地方，一般不冲突

```
pip install -r yolov5_requirements.txt
```

2、将yolov5s的模型放到weights中

3、模型转换

```
python export.py --weights ./weights/yolov5s.pt --img-size 640 --batch 1 --rknn_mode
```

如果成果则会在weights中生成yolov5s.onnx

4、将yolov5s.onnx转成yolov5s.rknn

## ONNX to RKNN

1、将生成的yolov5s.onnx放到onnx_to_rknn/examples/onnx/yolov5s中
cd onnx_to_rknn/examples/onnx/yolov5s
2、模型转换
python test.py
如果成功则会在目录下生成yolov5s.onnx，同时生成结果照片

## RKNN to deploy

5、将yolov5s.rknn部署到rk3399或其他芯片的板子上。
进入到rknn_to_deploy/examples/yolov5s目录中
执行
bash build-linux.sh
如果成功则生成install跟build两个文件夹
将install文件放到板子上，执行
./yolov5s_demo
可以看到在model生成了out.jpg
走到这一步，恭喜你已经完成模型的布署。
参考来源：https://github.com/mrwangwg123/my-rknn-yolov5
参考来源：https://github.com/ultralytics/yolov5
参考来源：https://github.com/airockchip/yolov5
参考来源：https://github.com/Dreamdreams8/yolov5s_rknn_deploy