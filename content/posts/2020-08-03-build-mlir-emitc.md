---
title: "MLIR EmitCをビルドする"
slug: "build-mlir-emitc"
subtitle:    ""
description: ""
date:        "2020-08-03"
author:      "Masahiro Hiramori"
image:       ""
tags:        ["MLIR"]
categories:  ["Tech"]
draft:       false
---

[C++コードを出力できるMLIR Dialect](https://github.com/iml130/mlir-emitc/)をビルドする。
[ONNX MLIRをビルドする]({{< ref "/posts/2020-07-27-building-onnx-mlir.md" >}})でMLIRをビルド済みとする。
MLIR(LLVM)のソースコードは`~/workspace/llvm-project`へ配置済み。

## ソースコード取得

```bash
$ cd ~/workspace
$ git clone https://github.com/iml130/mlir-emitc/
```

## 環境変数設定

```bash
export MLIR_DIR=$(pwd)/llvm-project/build/lib/cmake/mlir
export LLVM_EXTERNAL_LIT~$(pwd)/llvm-project/build/bin/llvm-lit
```

## ビルド

```bash
mkdir mlir-emitc/build && cd mlir-emitc/build
cmake -G Ninja -DCMAKE_C_COMPILER=clang -DCMAKE_CXX_COMPILER=clang++ .. \
  -DMLIR_DIR=$MLIR_DIR -DLLVM_EXTERNAL_LIT=$LLVM_EXTERNAL_LIT
cmake --build . --target check-emitc
```

テストエラーが出るが、一応ビルドはできた。

```bash
cmake --build . --target check-emitc
[12/13] Running the EmitC regression tests
-- Testing: 2 tests, 2 workers --
FAIL: EMITC :: Target/cpp-calls.mlir (1 of 2)
PASS: EMITC :: Dialect/EmitC/ops.mlir (2 of 2)
********************
Failed Tests (1):
  EMITC :: Target/cpp-calls.mlir


Testing Time: 0.11s
  Passed: 1
  Failed: 1
FAILED: test/CMakeFiles/check-emitc
cd /home/mshr/workspace/mlir-emitc/build/test && /home/mshr/workspace/llvm-project/build/./bin/llvm-lit /home/mshr/workspace/mlir-emitc/build/test
ninja: build stopped: subcommand failed.
```

`build/bin`ディレクトリに`emitc-opt`と`emitc-translate`が生成される。

```bash
ls bin
emitc-opt  emitc-translate
```
