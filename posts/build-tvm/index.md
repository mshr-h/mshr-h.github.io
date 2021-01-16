# TVM v0.7ビルド手順


## CAUTION

少し古い情報(2020/9頃)のため、2021/01/16時点では不要な情報の可能性が高いが、記録のために残しておく。

## ビルド

```bash
sudo apt-get update
sudo apt-get install -y python3 python3-dev python3-setuptools gcc libtinfo-dev zlib1g-dev build-essential cmake libedit-dev libxml2-dev ninja-build
```

```bash
cd ~/workspace
git clone https://github.com/llvm/llvm-project.git
cd llvm-project && git checkout llvmorg-10.0.1 && cd ..
```

TVMと[ONNX-MLIR](https://github.com/onnx/onnx-mlir)を共存するために、ONNX-MLIRが依存する[LLVM@1d01fc](https://github.com/llvm/llvm-project/tree/1d01fc)(未リリースのバージョン)を使ってTVMをビルドすると、ビルドエラーが発生する。(後述)

```bash
cd ~/workspace
git clone --recursive https://github.com/apache/incubator-tvm tvm
mkdir build
cp cmake/config.cmake build
cd build
```

```bash
vim config.cmake
```

```text
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

#--------------------------------------------------------------------
#  Template custom cmake configuration for compiling
#
#  This file is used to override the build options in build.
#  If you want to change the configuration, please use the following
#  steps. Assume you are on the root directory. First copy the this
#  file so that any local changes will be ignored by git
#
#  $ mkdir build
#  $ cp cmake/config.cmake build
#
#  Next modify the according entries, and then compile by
#
#  $ cd build
#  $ cmake ..
#
#  Then build in parallel with 8 threads
#
#  $ make -j8
#--------------------------------------------------------------------

#---------------------------------------------
# Backend runtimes.
#---------------------------------------------

# Whether enable CUDA during compile,
#
# Possible values:
# - ON: enable CUDA with cmake's auto search
# - OFF: disable CUDA
# - /path/to/cuda: use specific path to cuda toolkit
set(USE_CUDA OFF)

# Whether enable ROCM runtime
#
# Possible values:
# - ON: enable ROCM with cmake's auto search
# - OFF: disable ROCM
# - /path/to/rocm: use specific path to rocm
set(USE_ROCM OFF)

# Whether enable SDAccel runtime
set(USE_SDACCEL OFF)

# Whether enable Intel FPGA SDK for OpenCL (AOCL) runtime
set(USE_AOCL OFF)

# Whether enable OpenCL runtime
#
# Possible values:
# - ON: enable OpenCL with cmake's auto search
# - OFF: disable OpenCL
# - /path/to/opencl-sdk: use specific path to opencl-sdk
set(USE_OPENCL OFF)

# Whether enable Metal runtime
set(USE_METAL OFF)

# Whether enable Vulkan runtime
#
# Possible values:
# - ON: enable Vulkan with cmake's auto search
# - OFF: disable vulkan
# - /path/to/vulkan-sdk: use specific path to vulkan-sdk
set(USE_VULKAN OFF)

# Whether enable OpenGL runtime
set(USE_OPENGL OFF)

# Whether enable MicroTVM runtime
set(USE_MICRO ON)

# Whether to enable SGX runtime
#
# Possible values for USE_SGX:
# - /path/to/sgxsdk: path to Intel SGX SDK
# - OFF: disable SGX
#
# SGX_MODE := HW|SIM
set(USE_SGX OFF)
set(SGX_MODE "SIM")
set(RUST_SGX_SDK "/path/to/rust-sgx-sdk")

# Whether enable RPC runtime
set(USE_RPC ON)

# Whether to build the C++ RPC server binary
set(USE_CPP_RPC ON)

# Whether embed stackvm into the runtime
set(USE_STACKVM_RUNTIME OFF)

# Whether enable tiny embedded graph runtime.
set(USE_GRAPH_RUNTIME ON)

# Whether enable additional graph debug functions
set(USE_GRAPH_RUNTIME_DEBUG ON)

# Whether enable additional vm profiler functions
set(USE_VM_PROFILER ON)

# Whether enable uTVM standalone runtime
set(USE_MICRO_STANDALONE_RUNTIME ON)

# Whether build with LLVM support
# Requires LLVM version >= 4.0
#
# Possible values:
# - ON: enable llvm with cmake's find search
# - OFF: disable llvm
# - /path/to/llvm-config: enable specific LLVM when multiple llvm-dev is available.
set(USE_LLVM /home/ubuntu/workspace/llvm-project/build/bin/llvm-config)

#---------------------------------------------
# Contrib libraries
#---------------------------------------------
# Whether use BLAS, choices: openblas, atlas, apple
set(USE_BLAS none)

# Whether to use MKL
# Possible values:
# - ON: Enable MKL
# - /path/to/mkl: mkl root path
# - OFF: Disable MKL
# set(USE_MKL /opt/intel/mkl) for UNIX
# set(USE_MKL ../IntelSWTools/compilers_and_libraries_2018/windows/mkl) for WIN32
# set(USE_MKL <path to venv or site-packages directory>) if using `pip install mkl`
set(USE_MKL OFF)

# Whether use MKLDNN library, choices: ON, OFF, path to mkldnn library
set(USE_MKLDNN OFF)

# Whether use OpenMP thread pool, choices: gnu, intel
# Note: "gnu" uses gomp library, "intel" uses iomp5 library
set(USE_OPENMP none)

# Whether use contrib.random in runtime
set(USE_RANDOM ON)

# Whether use NNPack
set(USE_NNPACK OFF)

# Possible values:
# - ON: enable tflite with cmake's find search
# - OFF: disable tflite
# - /path/to/libtensorflow-lite.a: use specific path to tensorflow lite library
set(USE_TFLITE OFF)

# /path/to/tensorflow: tensorflow root path when use tflite library
set(USE_TENSORFLOW_PATH none)

# Required for full builds with TFLite. Not needed for runtime with TFLite.
# /path/to/flatbuffers: flatbuffers root path when using tflite library
set(USE_FLATBUFFERS_PATH none)

# Possible values:
# - OFF: disable tflite support for edgetpu
# - /path/to/edgetpu: use specific path to edgetpu library
set(USE_EDGETPU OFF)

# Whether use CuDNN
set(USE_CUDNN OFF)

# Whether use cuBLAS
set(USE_CUBLAS OFF)

# Whether use MIOpen
set(USE_MIOPEN OFF)

# Whether use MPS
set(USE_MPS OFF)

# Whether use rocBlas
set(USE_ROCBLAS OFF)

# Whether use contrib sort
set(USE_SORT ON)

# Whether use MKL-DNN (DNNL) codegen
set(USE_DNNL_CODEGEN OFF)

# Whether to use Arm Compute Library (ACL) codegen
# We provide 2 separate flags since we cannot build the ACL runtime on x86.
# This is useful for cases where you want to cross-compile a relay graph
# on x86 then run on AArch.
#
# An example of how to use this can be found here: docs/deploy/arm_compute_lib.rst.
#
# USE_ARM_COMPUTE_LIB - Support for compiling a relay graph offloading supported
#                       operators to Arm Compute Library. OFF/ON
# USE_ARM_COMPUTE_LIB_GRAPH_RUNTIME - Run Arm Compute Library annotated functions via the ACL
#                                     runtime. OFF/ON/"path/to/ACL"
set(USE_ARM_COMPUTE_LIB OFF)
set(USE_ARM_COMPUTE_LIB_GRAPH_RUNTIME OFF)

# Whether to build with Arm Ethos-N support
# Possible values:
# - OFF: disable Arm Ethos-N support
# - path/to/arm-ethos-N-stack: use a specific version of the
#   Ethos-N driver stack
set(USE_ETHOSN OFF)
# If USE_ETHOSN is enabled, use ETHOSN_HW (ON) if Ethos-N hardware is available on this machine
# otherwise use ETHOSN_HW (OFF) to use the software test infrastructure
set(USE_ETHOSN_HW OFF)

# Build ANTLR parser for Relay text format
# Possible values:
# - ON: enable ANTLR by searching default locations (cmake find_program for antlr4 and /usr/local for jar)
# - OFF: disable ANTLR
# - /path/to/antlr-*-complete.jar: path to specific ANTLR jar file
set(USE_ANTLR OFF)

# Whether use Relay debug mode
set(USE_RELAY_DEBUG ON)

# Whether to build fast VTA simulator driver
set(USE_VTA_FSIM OFF)

# Whether to build cycle-accurate VTA simulator driver
set(USE_VTA_TSIM OFF)

# Whether to build VTA FPGA driver (device side only)
set(USE_VTA_FPGA OFF)

# Whether use Thrust
set(USE_THRUST OFF)

# Whether to build the TensorFlow TVMDSOOp module
set(USE_TF_TVMDSOOP OFF)

# Whether to use STL's std::unordered_map or TVM's POD compatible Map
set(USE_FALLBACK_STL_MAP OFF)

# Whether to use hexagon device
set(USE_HEXAGON_DEVICE OFF)
set(USE_HEXAGON_SDK /path/to/sdk)

# Whether to use ONNX codegen
set(USE_TARGET_ONNX ON)

# Whether to compile the standalone C runtime.
set(USE_STANDALONE_CRT ON)
```

```bash
cmake .. -G Ninja
ninja
```

```bash
export TVM_HOME=/path/to/tvm
export PYTHONPATH=$TVM_HOME/python:${PYTHONPATH}
```

## ビルドエラーの詳細

[LLVM@1d01fc](https://github.com/llvm/llvm-project/tree/1d01fc)を使用してTVMをビルドすると下記エラーが発生する。

```
[17/85] Building CXX object CMakeFiles/tvm_objs.dir/src/target/llvm/codegen_llvm.cc.o
FAILED: CMakeFiles/tvm_objs.dir/src/target/llvm/codegen_llvm.cc.o 
/usr/bin/c++  -DBUILD_EXAMPLES -DDMLC_USE_FOPEN64=0 -DTVM_INDEX_DEFAULT_I64=1 -DTVM_LLVM_VERSION=120 -DTVM_THREADPOOL_USE_OPENMP=0 -DUSE_FALLBACK_STL_MAP=0 -DUSE_MICRO_STANDALONE_RUNTIME=1 -D_DEBUG -D_GNU_SOURCE -D__STDC_CONSTANT_MACROS -D__STDC_FORMAT_MACROS -D__STDC_LIMIT_MACROS -DDMLC_ENABLE_RTTI=0 -I../include -I../3rdparty/dlpack/include -I../3rdparty/dmlc-core/include -I../3rdparty/rang/include -I../3rdparty/compiler-rt -I../3rdparty/picojson -I/home/ubuntu/workspace/llvm-project/llvm/include -I/home/ubuntu/workspace/llvm-project/build/include -I../topi/include -std=c++14 -faligned-new -O2 -Wall -fPIC    -fno-rtti -MD -MT CMakeFiles/tvm_objs.dir/src/target/llvm/codegen_llvm.cc.o -MF CMakeFiles/tvm_objs.dir/src/target/llvm/codegen_llvm.cc.o.d -o CMakeFiles/tvm_objs.dir/src/target/llvm/codegen_llvm.cc.o -c ../src/target/llvm/codegen_llvm.cc
../src/target/llvm/codegen_llvm.cc: In member function ‘llvm::Value* tvm::codegen::CodeGenLLVM::CreateBroadcast(llvm::Value*, int)’:
../src/target/llvm/codegen_llvm.cc:480:82: error: ‘llvm::ElementCount::ElementCount(unsigned int, bool)’ is private within this context
  480 |       llvm::ConstantVector::getSplat(llvm::ElementCount(lanes, /*Scalable=*/false), zero);
      |                                                                                  ^
In file included from /home/ubuntu/workspace/llvm-project/llvm/include/llvm/IR/Type.h:24,
                 from /home/ubuntu/workspace/llvm-project/llvm/include/llvm/IR/DerivedTypes.h:23,
                 from /home/ubuntu/workspace/llvm-project/llvm/include/llvm/IR/Constants.h:31,
                 from /home/ubuntu/workspace/llvm-project/llvm/include/llvm/IR/Operator.h:19,
                 from /home/ubuntu/workspace/llvm-project/llvm/include/llvm/Analysis/TargetTransformInfo.h:24,
                 from ../src/target/llvm/llvm_common.h:33,
                 from ../src/target/llvm/codegen_llvm.h:49,
                 from ../src/target/llvm/codegen_llvm.cc:25:
/home/ubuntu/workspace/llvm-project/llvm/include/llvm/Support/TypeSize.h:39:3: note: declared private here
   39 |   ElementCount(unsigned Min, bool Scalable) : Min(Min), Scalable(Scalable) {}
      |   ^~~~~~~~~~~~
[22/85] Building CXX object CMakeFiles/tvm_objs.dir/src/runtime/rpc/rpc_endpoint.cc.o
ninja: build stopped: subcommand failed.
```

エラーメッセージを読むと、privateメソッドとして定義されている`llvm::ElementCount::ElementCount(unsigned int, bool)`を
クラス外から呼び出そうとしているのが原因。

実際にLLVMの[TypeSize.h@1d01fc](https://github.com/llvm/llvm-project/blob/1d01fc/llvm/include/llvm/Support/TypeSize.h#L39)を見ると、そのように定義されている。

```cpp
class ElementCount {
private:
  unsigned Min;  // Minimum number of vector elements.
  bool Scalable; // If true, NumElements is a multiple of 'Min' determined
                 // at runtime rather than compile time.

  /// Prevent code from using initializer-list contructors like
  /// ElementCount EC = {<unsigned>, <bool>}. The static `get*`
  /// methods below are preferred, as users should always make a
  /// conscious choice on the type of `ElementCount` they are
  /// requesting.
  ElementCount(unsigned Min, bool Scalable) : Min(Min), Scalable(Scalable) {}
```

コメントには、下記の`get`スタティックメソッドを推奨する旨が書いているが、TVM側がこの変更に追いついてないためビルドエラーが発生した。

[TypeSize.h@1d01fc](https://github.com/llvm/llvm-project/blob/1d01fc/llvm/include/llvm/Support/TypeSize.h#L77-L79)

```cpp
  static ElementCount get(unsigned Min, bool Scalable) {
    return {Min, Scalable};
  }
```

### `ElementCount(unsigned int, bool)`のアクセス修飾子の変更経緯を追いかける。

[b302561 (Oct 8, 2019, 9:53 PM GMT+9)](https://github.com/llvm/llvm-project/commit/b302561)で該当行を含むファイルが新規作成される。この時点で`ElementCount(unsigned Min, bool Scalable)`はpublicメソッドとして定義されている。

[TypeSize.h@b302561](https://github.com/llvm/llvm-project/blob/b302561/llvm/include/llvm/Support/TypeSize.h#L28-L29)

```cpp
class ElementCount {
public:
  unsigned Min;  // Minimum number of vector elements.
  bool Scalable; // If true, NumElements is a multiple of 'Min' determined
                 // at runtime rather than compile time.

  ElementCount(unsigned Min, bool Scalable)
  : Min(Min), Scalable(Scalable) {}
```

[a407ec (Aug 20, 2020, 2:26 AM GMT+9)](https://github.com/llvm/llvm-project/commit/a407ec)で`ElementCount(unsigned Min, bool Scalable)`がpublicからprivateへ変更される。(これが1d01fc時点のコード)

[TypeSize.h@a407ec](https://github.com/llvm/llvm-project/blob/a407ec/llvm/include/llvm/Support/TypeSize.h#L28-L35)

```cpp
class ElementCount {
private:
  /// Prevent code from using initializer-list contructors like
  /// ElementCount EC = {<unsigned>, <bool>}. The static `get*`
  /// methods below are preferred, as users should always make a
  /// conscious choice on the type of `ElementCount` they are
  /// requesting.
  ElementCount(unsigned Min, bool Scalable) : Min(Min), Scalable(Scalable) {}
```

`llvmorg-10.0.1`はb302561のコードを使用している(=publicメソッドとして定義されている)ため、問題なくビルドできる。

