## Introduction 
CentOS 7 as one of the most popular Linux distributions, is widely adopted by developers for software development and deployment.

However, CentOS 7 default GCC 4.8.5 and CMake 2.8.12 does not meet the OpenVINO [software requirement](https://github.com/openvinotoolkit/openvino/wiki/BuildingForLinux#software-requirements) for building on Linux system.

In this guide, we will show how to build OpenVINO on CentOS 7.6 (64 bit) using custom GCC 8.3.1, CMake 3.23.2, and Python 3.7. 

User may choose other GCC, CMake, and Python version that meets [software requirement](https://github.com/openvinotoolkit/openvino/wiki/BuildingForLinux#software-requirements) for building OpenVINO on Linux system.

## 1. Install system dependency and setup environment
### 1.1. Setup GCC 8.3.1
Install GCC 8.3.1 via devtoolset-8 
```sh
sudo yum update -y && sudo yum install -y centos-release-scl epel-release
sudo yum install -y devtoolset-8 git patchelf
```
Enable devtoolset-8 and check current gcc version
```sh
source /opt/rh/devtoolset-8/enable
gcc -v 
```

### 1.2. Setup CMake 3.23.2
Download and extract pre-built CMake 3.23.2
```sh
wget https://cmake.org/files/v3.23/cmake-3.23.2-linux-x86_64.tar.gz
tar -xvf cmake-3.23.2-linux-x86_64.tar.gz
```
Add cmake path to the PATH environment variable and check current cmake version
```sh
export PATH=$PWD/cmake-3.23.2-linux-x86_64/bin:$PATH
cmake --version
```
### 1.3. Setup python virtual environment via miniconda3
Download and install miniconda3
```sh
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
sudo chmod +x Miniconda3-latest-Linux-x86_64.sh
./Miniconda3-latest-Linux-x86_64.sh
```
Create and enable python virtual environment via miniconda3
```sh
conda create -n ov_py37 python=3.7 -y
conda activate ov_py37
```

## 2. Build OpenVINO with CMake 3.23.2 and GCC 8.3.1
Clone openvino and update submodule
```sh
git clone https://github.com/openvinotoolkit/openvino.git
cd openvino && git submodule update --init --recursive
```
Install python dependency for building python wheels
```sh
python3 -m pip install -U pip 
python3 -m pip install -r ./src/bindings/python/src/compatibility/openvino/requirements-dev.txt
python3 -m pip install -r ./src/bindings/python/wheel/requirements-dev.txt
```
Create build and openvino_dist directory
```sh
mkdir build && mkdir openvino_dist && cd build
```
Build OpenVINO with CMake
```sh
cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=../openvino_dist \
      -DENABLE_OPENCV=OFF -DENABLE_INTEL_GNA=OFF -DENABLE_INTEL_MYRIAD_COMMON=OFF \
      -DTREAT_WARNING_AS_ERROR=OFF -DENABLE_PYTHON=ON -DENABLE_WHEEL=ON \
      -DPYTHON_EXECUTABLE=`which python3.7` \
      -DPYTHON_LIBRARY=~/miniconda3/envs/ov_py37/lib/libpython3.7m.so \
      -DPYTHON_INCLUDE_DIR=~/miniconda3/envs/ov_py37/include/python3.7m  ..
make --jobs=$(nproc --all)
make install
```
Install built python wheel for OpenVINO runtime and OpenVINO-dev tools
```sh
python3 -m pip install openvino-dev --find-links <openvino_repo>/openvino_dist/tools
```

## 3. Quick test for built openvino runtime and openvino-dev tools
Create model directory and install model optimizer dependency
```sh
mkdir ~/ov_models
pip install onnx==1.11.0 protobuf==3.19.4 openvino-dev[pytorch]
```
Download resnet50 pytorch model via omz_downloader
```sh
omz_downloader --name resnet-50-pytorch -o ~/ov_models/
```
Convert resnet50 pytorch model to OpenVINO FP32 IR via omz_converter
```sh
omz_converter --name resnet-50-pytorch -o ~/ov_models/ -d ~/ov_models/
```
Run benchmark app with resnet50 FP32 IR model on CPU
```sh
benchmark_app -m ~/ov_models/public/resnet-50-pytorch/FP32/resnet-50-pytorch.xml -d CPU
```