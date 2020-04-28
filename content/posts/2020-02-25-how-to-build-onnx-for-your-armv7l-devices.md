---
title: "How to build onnx/onnx for your ARMv7l devices"
slug: "how-to-build-onnx-for-your-armv7l-devices"
subtitle:    ""
description: "In this blog post, I'll explain how to build onnx/onnx for ARMv7l devices."
date:        "2020-02-25"
author:      "Masahiro Hiramori"
tags:        ["ARM", "Linux", "ONNX Runtime"]
categories:  ["Tech" ]
draft:       false
---

[microsoft/onnxruntime](https://github.com/microsoft/onnxruntime) provides build instruction for ARMv7l python wheel which requires [onnx/onnx](https://github.com/onnx/onnx). But it doesn't provide binary package nor build instruction for ARMv7l. So I've written a dockerfile to build [onnx/onnx](https://github.com/onnx/onnx) python wheel.


# Build instruction

## 1. Create a build directory

```
mkdir build-onnx
cd build-onnx
```

## 2. Save the dockerfile on into the build directory

- `Dockerfile.arm32v7`

```
FROM balenalib/raspberrypi3-python:latest-stretch-build

ARG REPO_URL=https://github.com/onnx/onnx
ARG BRANCH=master

#Enforces cross-compilation through Qemu
RUN [ "cross-build-start" ]

RUN install_packages \\
    sudo \\
    build-essential \\
    cmake \\
    git \\
    python3 \\
    python3-pip \\
    python3-dev \\
    libprotoc-dev \\
    protobuf-compiler

RUN pip3 install --upgrade pip setuptools wheel
RUN pip3 install numpy

# Prepare onnx Repo
WORKDIR /code
RUN git clone --single-branch --branch ${BRANCH} --recursive ${REPO_URL} onnx

# Start the basic build
WORKDIR /code/onnx
RUN python3 setup.py bdist_wheel

# Build Output
RUN realpath /code/onnx/dist/onnx-*.whl

RUN [ "cross-build-end" ]
```

## 3. Run docker build

It will take several hours depending on your machine specs.

```
docker build -t onnx-arm32v7 -f Dockerfile.arm32v7 .
```

## 4. Note the full path of the `.whl` file

It will print the path something like this.

```
Step 12/13 : RUN realpath /code/onnx/dist/onnx-*.whl
---> Running in ded3b4302e29
/code/onnx/dist/onnx-1.6.0-cp35-cp35m-linux_armv7l.whl
```

## 5. Copy the Python wheel from the docker image

```
docker create -ti --name onnx_temp onnx-arm32v7 bash
docker cp onnx_temp:/code/onnx/dist/onnx-1.6.0-cp35-cp35m-linux_armv7l.whl .
docker rm -fv onnx_temp
```

## 6. Copy the wheel file to your ARMv7l device

example:

```
scp onnx-1.6.0-cp35-cp35m-linux_armv7l.whl pi@RaspberryPi:/home/pi/
```

## 7. On device, install the wheel file

```
pip install onnx-1.6.0-cp35-cp35m-linux_armv7l.whl
```