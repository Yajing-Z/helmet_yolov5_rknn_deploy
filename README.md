# Helmet-YOLOv5 Model Deploy to RK3568

## Helmet RKNN Model to deploy in RK3568

这里我们直接进行将已生成好的helmet-640-640.rknn模型，部署到rk3568或其他芯片的板子上的过程

1. Install requirements (这里的以Debian10，Python3.7环境为例)

```bash
sudo apt update

#安装其他python工具
sudo apt-get install python3-dev python3-pip gcc

#安装相关依赖和软件包
sudo apt-get install -y python3-opencv
sudo apt-get install -y python3-numpy
sudo apt -y install python3-setuptools
sudo pip3 install wheel
```

2. Toolkit Lite2工具安装及python依赖环境安装：

```bash
git clone https://github.com/harperjuanl/helmet_yolov5_rknn_deploy.git
cd helmet_yolov5_rknn_deploy/rknn_to_deploy/
sudo pip3 install rknn_toolkit_lite2-1.4.0-cp37-cp37m-linux_aarch64.whl
sudo pip3 install -r requirements.txt -i https://mirrors.aliyun.com/pypi/simple/

# 检查Toolkit Lite2是否安装成功
pip3 list | grep rknn-toolkit-lite2
rknn-toolkit-lite2           1.4.0
```

3. Model deployment and inference test

```bash
cd rknn_to_deploy/examples/yolov5s
# you can also upload images to the folder and choose the image you want to detect
python3 inference.py --img test.jpg  
```

图片检测结果可以在model里生成了test_result.jpg看到

![image](https://github.com/harperjuanl/helmet_yolov5_rknn_deploy/blob/main/imgs/rknn_inference.png)

## Helmet Model convert to RKNN Model in PC platform

You can also try the progress of AI model convert to rknn model in your ubuntu machine (the generated rknn model is needed in the above rk3568 chip)

### Prerequisite

1. 安装基础依赖（我们这里使用的环境是PC ubuntu20.04，ubuntu20.04默认是安装了python3.8.10）

```bash
sudo apt update
sudo apt-get install python3-dev python3-pip python3.8-venv gcc

#安装相关库和软件包
sudo apt-get install libxslt1-dev zlib1g-dev libglib2.0 libsm6 \
libgl1-mesa-glx libprotobuf-dev gcc
```

2. 安装rknn-toolkit2

```bash
#创建目录，由于测试使用的ubuntu20.04已经安装的包可能和安装运行RKNN-Toolkit2所需的包版本不同,为避免其他问题，这里使用python venv隔离环境
mkdir project-Toolkit2 && cd project-Toolkit2
python3 -m venv .toolkit2_env
# 激活进入环境
source .toolkit2_env/bin/activate

#拉取源码，或者复制RKNN-Toolkit2到该目录
git clone https://github.com/rockchip-linux/rknn-toolkit2.git

#使用pip3安装包时可能很慢，设置下源
pip3 config set global.index-url https://mirror.baidu.com/pypi/simple

#安装依赖库，根据rknn-toolkit2\doc\requirements_cp38-1.4.0.txt
pip3 install numpy
pip3 install -r doc/requirements_cp38-1.4.0.txt

#安装rknn_toolkit2
pip3 install packages/rknn_toolkit2-1.4.0_22dcfef4-cp38-cp38-linux_x86_64.whl

```

3. 检查 RKNN-Toolkit 是否安装成功

```bash
$ python3 
Python 3.8.10 (default, Jun 22 2022, 20:18:18)
[GCC 9.4.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> from rknn.api import RKNN
>>>
```

### Helmet Yolov5 to ONNX

将helmet.pt转成helmet.onnx

```
cd helmet_yolov5_rknn_deploy/yolov5s-to-onnx
```

1. 安装依赖环境

```
pip3 install -r yolov5_requirements.txt
```

2. 将helmet-yolov5的模型放到weights中，模型转换

```
python3 export.py --weights ./weights/helmet.pt --img-size 640 --batch 1 --rknn_mode
```

如果成功，则会在weights中生成helmet.onnx

### ONNX to RKNN

将helmet.onnx转成helmet.rknn

1. 将生成的helmet.onnx放到onnx_to_rknn/examples/onnx/yolov5s中

```bash
cd onnx_to_rknn/examples/onnx/yolov5s
```

2. 进行模型转换以及图片测试

```bash
python3 test.py
```

如果成功，则会在目录下生成helmet-640-640.rknn，同时生成测试结果照片。而helmet-640-640.rknn会在RK3568板子上部署helmet detection model时用到

![image](https://github.com/harperjuanl/helmet_yolov5_rknn_deploy/blob/main/imgs/rknn_convert.png)

## Reference

参考来源：https://github.com/rockchip-linux/rknn-toolkit2

参考来源：https://github.com/ultralytics/yolov5

参考来源：https://github.com/Dreamdreams8/yolov5s_rknn_deploy
