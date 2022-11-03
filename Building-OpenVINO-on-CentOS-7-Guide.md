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

## 4. Troubleshooting
For general troubleshooting steps and issues, see [Troubleshooting Guide for OpenVINO Installation](https://docs.openvino.ai/2022.2/openvino_docs_get_started_guide_troubleshooting.html). The following provide explanations to error messages during building OpenVINO on LINUX platforms.

### 4.1 Build with gflags error

```sh
/usr/local/lib/libgflags_nothreads.a(gflags.cc.o): relocation R_X86_64_32 against '.bss' can not be used when making a PIE object; recompile with -fPIC

/usr/local/lib/libgflags_nothreads.a(gflags reporting.cc.o): relocation R_X86_64_32 against symbol '__pthread key create@@GLIBC 2.2.5' can not be used when making a PIE object; recompile with -fPIC

/usr/local/lib/libgflags_nothreads.a(gflags completions.cc.o0): relocation R_X86 64 32S against symbol '__ZNSs4 Rep20 S empty _rep_storageE@@GLIBCXX_3.4' can not be used when making a PIE object; recompile with -fPIC
```
#### Root cause:
- This issue probably appears in OpenVINO 2022.1 and 2022.2 versions. OpenVINO requires third-party's libraries built with -fPIC option, which generates position-independent code (PIC) to be used in a shared library. For more detailed PIC usage and limitation, please check with [user guide of GCC compiler](https://gcc.gnu.org/onlinedocs/gcc/Code-Gen-Options.html#Code-Gen-Options).
- User will not have this issue in master branch that OpenVINO allows to use system-installed gflags in Fedora and CentOS.

To solve the above issue:
- Make sure you use OpenVINO submodule to download third-party's libraries to compile and generate local static libs for linking.
- Use 'gflags' as an example, OpenVINO will use sources in `openvino/thirdparty/gflags/` to build new static lib to use.

#### Solution 1 for OpenVINO version before 2022.3:

1. Remove CentOS 7 system installed `gflags` by running `yum remove gflags-devel` and any other installation of `gflags`.
2. Make sure there's no system installed gflags lib, verify that `gflags` was removed as follow:
```sh
echo "$(ldconfig -p | grep libgflags.so | tr ' ' '\n' | grep /)"

rpm -ql gflags-devel
```

3. Build OpenVINO package

Run CMake to set OpenVINO build configuration. The package will build gflags in `thridparty` folder and set -Dgflags_DIR variable to the locally installed lib in OpenVINO package. Expected configuration log as below:

`The name gflags is an ALIAS for gflags_nothreads_static. It will be exported to the OpenVINODeveloperPackage with the original name.`

4. Then continue to compile and install the OpenVINO.

#### Solution 2 for OpenVINO version since 2022.3:

1. Install `gflags` in system by running `yum install gflags-devel`. Two dynamic library `libgflags.so` and `libgflags_nothreads.so` will be installed in `/usr/lib64/`.
2. Build OpenVINO package

Run CMake to set OpenVINO build configuration and build. The package will use system-installed gflags lib. Expected configuration log as below: 
`gflags (2.1.1) is found at /usr/lib64/cmake/gflags`

3. Then continue to compile and install the OpenVINO.
