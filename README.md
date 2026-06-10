# 测试流程说明

## 编译命令

### 交叉编译

工具链地址：

https://nexus.bianbu.xyz/#browse/browse:toolchain:llvm-gcc%2Fspacemit-toolchain-linux-glibc-x86_64-v1.0.4.tar.xz

```Shell
export PATH=path/to/toolchain:$PATH
export CXX=riscv64-unknown-linux-gnu-g++
export CC=riscv64-unknown-linux-gnu-gcc
git clone https://github.com/opencv/opencv.git
```

在opencv同级目录下，执行：

```Shell
mkdir build
cd build
```

带RVV:

```Shell
cmake -DCMAKE_TOOLCHAIN_FILE="../opencv/platforms/linux/riscv64-gcc.toolchain.cmake" -DCMAKE_CXX_FLAGS="-march=rv64imafdcv_zicbom_zicboz_zicntr_zicond_zicsr_zifencei_zihintpause_zihpm_zfh_zfhmin_zca_zcd_zba_zbb_zbc_zbs_zkt_zve32f_zve32x_zve64d_zve64f_zve64x_zvfh_zvfhmin_zvkt" \
-DCMAKE_C_FLAGS="-march=rv64imafdcv_zicbom_zicboz_zicntr_zicond_zicsr_zifencei_zihintpause_zihpm_zfh_zfhmin_zca_zcd_zba_zbb_zbc_zbs_zkt_zve32f_zve32x_zve64d_zve64f_zve64x_zvfh_zvfhmin_zvkt" \
-DWITH_OPENCL=OFF \
../opencv/
cmake --build .
```

禁用RVV:

```Shell
cmake -DCMAKE_TOOLCHAIN_FILE="../opencv/platforms/linux/riscv64-gcc.toolchain.cmake" \                        
-DCMAKE_CXX_FLAGS="-march=rv64imafdcv_zicbom_zicboz_zicntr_zicond_zicsr_zifencei_zihintpause_zihpm_zfh_zfhmin_zca_zcd_zba_zbb_zbc_zbs_zkt_zve32f_zve32x_zve64d_zve64f_zve64x_zvfh_zvfhmin_zvkt" \
-DCMAKE_C_FLAGS="-march=rv64imafdcv_zicbom_zicboz_zicntr_zicond_zicsr_zifencei_zihintpause_zihpm_zfh_zfhmin_zca_zcd_zba_zbb_zbc_zbs_zkt_zve32f_zve32x_zve64d_zve64f_zve64x_zvfh_zvfhmin_zvkt" \
-DWITH_OPENCL=OFF -DCPU_BASELINE=  \
../opencv/
cmake --build .
```

### 本地直接编译

```
cd ~
git clone https://github.com/opencv/opencv.git 
git clone https://github.com/opencv/opencv_contrib.git

mkdir build_rvv && cd build_rvv

cmake -D CMAKE_BUILD_TYPE=Release \
      -D CMAKE_INSTALL_PREFIX=./install \
      -D CMAKE_CXX_FLAGS=-march=rv64gcv \
      -D OPENCV_EXTRA_MODULES_PATH=../opencv_contrib/modules \
      -D BUILD_JPEG=ON \
      -D BUILD_opencv_python3=OFF \
      -D BUILD_EXAMPLES=OFF ../opencv
```



## 测试命令

测试数据执行以下命令下载：

```Shell
git clone https://github.com/opencv/opencv_extra.git
cd ~/build_rvv/bin
export sub_dir=results_k1_rvv # 根据需要更改
export OPENCV_TEST_DATA_PATH=~/opencv_extra/testdata
export LD_LIBRARY_PATH=../lib:$LD_LIBRARY_PATH
mkdir $sub_dir
```

测试命令：

```Shell
./opencv_perf_core --perf_threads=4 --gtest_output=json:$sub_dir/opencv_core.json
./opencv_perf_calib3d --perf_threads=4 --gtest_output=json:$sub_dir/opencv_calib3d.json
./opencv_perf_imgproc --perf_threads=4 --gtest_output=json:$sub_dir/opencv_imgproc.json --gtest_filter='-*distanceTransform_NeedLabels/*'
./opencv_perf_features2d --perf_threads=4 --gtest_output=json:$sub_dir/opencv_features2d.json --gtest_filter='-*batchDistance_8U/*:*batchDistance_Dest_32S/*:*batchDistance_L2/*'
```

使用数据分析脚本时，需要让脚本和生成的结果文件夹同路径：
