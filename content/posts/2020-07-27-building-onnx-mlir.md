---
title: "ONNX MLIRをビルドする"
slug: "building-onnx-mlir"
subtitle:    ""
description: ""
date:        "2020-07-27"
author:      "Masahiro Hiramori"
image:       ""
tags:        ["ONNX", "MLIR"]
categories:  ["Tech"]
draft:       false
---

[ONNX MLIR](https://github.com/onnx/onnx-mlir)をビルドする手順。基本はGitHubのREADMEに記載の手順と同じ。

検証環境は

- Ryzen 5 1600
- Ubuntu 20.04 on WSL2
- ONNX MLIR commit id ac67900

## ビルドツール導入

ビルドに必要なツール群を導入する。
Ubuntu 18.04の場合、`libprotoc-dev`が古くてビルドできないため、別途手動で導入する必要がある。

```bash
sudo apt install protobuf-compiler build-essential cmake libprotoc-dev \
     ninja-build libncurses python3 python3-dev python-is-python3
```

## MLIRのビルド

カレントディレクトリは`~/workspace`とする。

LLVM内にあるMLIRを取得し、ONNX MLIRのビルドが確認できているブランチをチェックアウトする。

```BASH
cd ~/workspace
git clone https://github.com/llvm/llvm-project.git
cd llvm-project && git checkout 1d01fc100bb5bef5f5eaf92520b2e52f64ee1d6e && cd ..
```

ビルドする。Ryzen 5 1600で20分程度。

```BASH
mkdir llvm-project/build
cd llvm-project/build
cmake -G Ninja ../llvm \
   -DLLVM_ENABLE_PROJECTS=mlir \
   -DLLVM_BUILD_EXAMPLES=ON \
   -DLLVM_TARGETS_TO_BUILD="host" \
   -DCMAKE_BUILD_TYPE=Release \
   -DLLVM_ENABLE_ASSERTIONS=ON \
   -DLLVM_ENABLE_RTTI=ON

cmake --build .
cmake --build . --target check-mlir
```

## ONNX MLIRのビルド

onnx-mlirのソースコードを取得する。

```bash
cd ~/workspace
git clone --recursive https://github.com/onnx/onnx-mlir.git
```

ビルドする。Ryzen 5 1600で5分程度。

```bash
export LLVM_PROJ_SRC=$(pwd)/llvm-project/
export LLVM_PROJ_BUILD=$(pwd)/llvm-project/build

mkdir onnx-mlir/build && cd onnx-mlir/build
cmake -G Ninja ..
cmake --build .

export LIT_OPTS=-v
cmake --build . --target check-onnx-lit
```
