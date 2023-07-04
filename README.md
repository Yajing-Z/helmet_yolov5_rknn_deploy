# Deploy Helmet-YOLOv5 Model on RK3568, RK1808(NPU)

- [Deploy Helmet-YOLOv5 Model on RK3568, RK1808(NPU)](#deploy-helmet-yolov5-model-on-rk3568-rk1808npu)
  - [Deploy Helmet RKNN Model](#deploy-helmet-rknn-model)
    - [Deploy Helmet RKNN Model on RK3568](#deploy-helmet-rknn-model-on-rk3568)
      - [TroubleShooting](#troubleshooting)
    - [Deploy Helmet RKNN Model on RK1808(NPU)](#deploy-helmet-rknn-model-on-rk1808npu)
  - [Helmet Model convert to RKNN Model in PC platform](#helmet-model-convert-to-rknn-model-in-pc-platform)
    - [RK3568: Helmet Model convert to RKNN Model](#rk3568-helmet-model-convert-to-rknn-model)
      - [Prerequisite](#prerequisite)
      - [Helmet Yolov5 to ONNX](#helmet-yolov5-to-onnx)
      - [ONNX to RKNN](#onnx-to-rknn)
    - [RK1808(NPU): Helmet Model onnx convert to RKNN Model](#rk1808npu-helmet-model-onnx-convert-to-rknn-model)
      - [Prerequisite](#prerequisite-1)
      - [Helmet Yolov5 to ONNX](#helmet-yolov5-to-onnx-1)
      - [ONNX to RKNN](#onnx-to-rknn-1)
  - [Reference](#reference)


## Deploy Helmet RKNN Model

### Deploy Helmet RKNN Model on RK3568

这里我们直接进行将已生成好的helmet-640-640.rknn模型，部署到rk3568或其他芯片的板子上的过程

1. Install requirements (以Debian10，Python3.7环境为例)

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
cd helmet_yolov5_rknn_deploy/rknn_to_deploy/packages
sudo pip3 install numpy-1.21.6-cp37-cp37m-manylinux_2_17_aarch64.manylinux2014_aarch64.whl
sudo pip3 install onnxruntime-1.14.0-cp37-cp37m-manylinux_2_17_aarch64.manylinux2014_aarch64.whl
sudo pip3 install torch-1.13.1-cp37-cp37m-manylinux2014_aarch64.whl
sudo pip3 install rknn_toolkit_lite2-1.4.0-cp37-cp37m-linux_aarch64.whl

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

![image](https://github.com/Yajing-Z/helmet_yolov5_rknn_deploy/blob/main/imgs/rknn_inference.png)

#### TroubleShooting

The max version of glibc package in RK3568 chip is 2.28, which maybe is incompatible for the rknn model. If you also have the problem, you can follow the steps to manually upgrade the package

```bash
# check the glibc version
ldd --version

# Download libc6_2.29-0ubuntu1_arm64.deb from https://launchpad.net/ubuntu/+source/glibc/2.29-0ubuntu1/+build/16416023
dpkg -X ./libc6_2.29-0ubuntu1_arm64.deb  # 解压该文件
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/linaro/tmp/ext/lib/aarch64-linux-gnu

# check the updated glibc version
ldd --version
```

### Deploy Helmet RKNN Model on RK1808(NPU)

1. Enter RRK1808 from RK3568J
   
```bash
# ssh RK3568J first

# 查看 NIC 的当前配置，请使用 ifconfig命令 in the RK3568J
sudo ifconfig

# 要为 NIC 分配静态 IP 地址和网络掩码, enx10dcb69f302e 是RK1808虚拟网口
sudo ifconfig enx10dcb69f302e 192.168.180.1 netmask 255.255.255.0 up

# 设置路由转发，使得RK1808可以访问外网
#设置本地ipv4转发
sudo echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
sudo sysctl -p
#路由转发

# check alternatives to iptables-legacy
$ update-alternatives --config iptables

sudo iptables -t nat -L # List the rules in a chain or all chains

sudo iptables  -t  nat  -A POSTROUTING -o eth1 -j MASQUERADE
#其中eth1 需要修改为对应的以太网标识码
# 完成上述命令后，通过 sudo iptables -t nat -L 查看可以看到多了一行：MASQUERADE  all  --  anywhere             anywhere

# ssh to RK1808 from RK3568J
linaro@linaro-alip:~$ ssh toybrick@192.168.180.8
toybrick@192.168.180.8's password:
Linux debian10.toybrick 4.4.194 #7 SMP PREEMPT Tue Oct 27 15:35:44 CST 2020 aarch64
...

```

2. Install depencies

```bash
pip install torch==1.6.0
pip install opencv-python==4.4.0.46
pip install Pillow==5.3.0
```

3. Toolkit Lite 1.4.0 has installed on RK1808
```bash
# 在 RK1808M0计算棒上
# rknn-toolkit 版本是 1.4.0
toybrick@debian10:~$ pip list | grep rknn
rknn-toolkit-lite 1.4.0
```

1. Model deployment and inference test
   
```bash
cd rknn_to_deploy/examples/yolov5s-rk1808
# you can also upload images to the folder and choose the image you want to detect
python helmet_inference_rk1808.py --img test.jpg  
```
图片检测结果可以在model里生成了test_result.jpg看到, just take 0.06s for model inference:

![image](https://github.com/Yajing-Z/helmet_yolov5_rknn_deploy/blob/main/imgs/inference_result_rk1808.png)

## Helmet Model convert to RKNN Model in PC platform

For the convertment of the RKNN model from [rockchip-linux](https://github.com/rockchip-linux/rknn-toolkit), need to know:

- **RK1808**/RK1806/RV1109/RV1126: https://github.com/rockchip-linux/rknn-toolkit

- For RK3566/**RK3568**/RK3588/RK3588S/RV1103/RV1106, please refer to:
https://github.com/rockchip-linux/rknn-toolkit2

From [rockchip-linux_pytorch_yolov5s_example](https://github.com/rockchip-linux/rknn-toolkit/tree/master/examples/pytorch/yolov5), need to know that require upgrading the **rknn_toolkit version to 1.7.1** to load the yolov5 pytorch model.


### RK3568: Helmet Model convert to RKNN Model

You can also try the progress of AI model convert to rknn model in your ubuntu machine (the generated rknn model is needed in the above rk3568 chip)

#### Prerequisite

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

#### Helmet Yolov5 to ONNX

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

#### ONNX to RKNN

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

![image](https://github.com/Yajing-Z/helmet_yolov5_rknn_deploy/blob/main/imgs/rknn_convert.png)

### RK1808(NPU): Helmet Model onnx convert to RKNN Model

You can also try the progress of AI model convert to rknn model in your ubuntu machine.

#### Prerequisite

1. 安装基础依赖（我们这里使用的环境是PC ubuntu20.04，ubuntu20.04安装了python3.6.14）

```bash
# Apply for the Nimbus Ubuntu20.04 VM in the DBC
/mts/git/bin/nimbus deploy ovf atlas-ubuntu-vm-7 http://sc-prd-rdops-templates.eng.vmware.com/nimbus-templates/atlas-ubuntu-20-4/atlas-ubuntu-20-04/atlas-ubuntu-20-04.ovf --cpus=16 --memory 32768

#Logging on to the VM
user name/password: vmware/B1gd3m0z
$ ssh vmware@<your_vm_ip>

# Run the below command in the VM
sudo apt install libsqlite3-dev

# Install Python 3.6.14
# 从官网下载https://www.python.org/downloads/
wget https://www.python.org/ftp/python/3.6.14/Python-3.6.14.tgz
tar -xzvf Python-3.6.14.tgz

# build
cd Python-3.6.14
sudo ./configure --prefix=/usr/local/python3 --enable-loadable-sqlite-extensions
sudo make install

# Python 3.6 is installed now.

# Installed virtualenv 
sudo /usr/local/python3/bin/python3 -m pip install virtualenv
virtualenv rknnenv
$ python
Python 3.6.14 (default, Jul  3 2023, 03:42:21)
[GCC 9.3.0] on linux
Type "help", "copyright", "credits" or "license
>>>
```

2. 安装rknn-toolkit-1.7.1

```bash
# Install rknn_toolkit-1.7.1 on the python3.6 virtualenv

# Download url: https://github.com/rockchip-linux/rknn-toolkit/releases/tag/v1.7.1
pip install rknn_toolkit-1.7.1-cp36-cp36m-linux_x86_64.whl

```

3. 检查 RKNN-Toolkit 是否安装成功
   
```bash
$ python
Python 3.6.14 (default, Jul  3 2023, 03:42:21)
[GCC 9.3.0] on linux
Type "help", "copyright", "credits" or "license
>>>from rknn.api import RKNN
>>>
```

#### Helmet Yolov5 to ONNX

Refer to `Helmet Yolov5 to ONNX` of `RK3568: Helmet Model convert to RKNN Model`. 

Thus we directly use the [onnx model (helmet.onnx)](https://github.com/Yajing-Z/helmet_yolov5_rknn_deploy/blob/main/onnx_to_rknn/examples/onnx/yolov5s/helmet.onnx)

#### ONNX to RKNN

将helmet.onnx转成helmet.rknn

Refer to `ONNX to RKNN` of `RK3568: Helmet Model convert to RKNN Model`. 

You can directly use the [onnx2rknn.py](https://github.com/Yajing-Z/helmet_yolov5_rknn_deploy/blob/main/onnx_to_rknn/examples/onnx/yolov5s/onnx2rknn.py), just need to add the `target_platform = 'rk1808'`, as below:

```bash
        # Run the below if you plan to run rknn model on the RK1808
        rknn.config(mean_values=[[0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0]],
                   std_values=[[255.0, 255.0, 255.0, 255.0, 255.0, 255.0, 255.0, 255.0, 255.0, 255.0, 255.0, 255.0]],
                   batch_size=opt.batch_size,
                   target_platform = 'rk1808')
```

## Reference

参考来源：https://github.com/rockchip-linux/rknn-toolkit2

参考来源：https://github.com/ultralytics/yolov5

参考来源：https://github.com/Dreamdreams8/yolov5s_rknn_deploy

参考来源：https://github.com/rockchip-linux/rknn-toolkit
