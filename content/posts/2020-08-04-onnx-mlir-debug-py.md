---
title: "ONNX MLIRに付属のdebug.pyを動かす"
slug: "onnx-mlir-debug-py"
subtitle:    ""
description: ""
date:        "2020-08-04"
author:      "Masahiro Hiramori"
image:       ""
tags:        ["ONNX", "MLIR"]
categories:  ["Tech"]
draft:       false
---

[ONNX MLIR](https://github.com/onnx/onnx-mlir)に付属の[debug.py](https://github.com/onnx/onnx-mlir/blob/master/utils/debug.py)を動かす手順。
`.so`ファイルの実行手順を解析するための事前準備。

## 前提条件

- 環境
  - macOS Catalina 10.15.6
  - Python 3.7.5 (pyenvで導入)
  - ONNX MLIR commit id dbe0d734b5687e0aa7da911684912163cea07bd2
- ONNX MLIRをビルド済み([ONNX MLIRをビルドする]({{< ref "/posts/2020-07-27-building-onnx-mlir.md" >}}))
  - ビルドディレクトリは`$HOME/workspace/onnx-mlir/build`とする
- 作業ディレクトリは`$HOME/workspace/debug_test`とする

## 環境変数の設定

`ONNX_MLIR_HOME`にONNX MLIRのビルドディレクトリを設定する。

```bash
export ONNX_MLIR_HOME=$HOME/workspace/onnx-mlir/build
```

## 準備

`debug.py`を作業ディレクトリにコピーする。
ONNXモデルファイルの例としてmnistをダウンロードする。

```bash
cp $HOME/workspace/onnx-mlir/utils/debug.py $HOME/workspace/debug_test/
wget https://github.com/onnx/models/raw/master/vision/classification/mnist/model/mnist-8.onnx
```

Pythonのonnxパッケージを導入する。

```bash
pip3 install onnx
```

## 実行

`debug.py`を見る限り、ONNX MLIRでONNXモデルファイルを`.so`へ変換し、実行するようだ。

```bash
python3 debug.py mnist-8.onnx
Temporary directory has been created at /var/folders/mg/v57g_1jj3s52wgwtd61rt8t80000gn/T/tmp1megvpue
%8 = "krnl.getref"(%7, %c0_i64) : (memref<10368xi8>, i64) -> memref<1x8x18x18xf32>
Verifying value of Parameter193_reshape1
Verifying value of Convolution28_Output_0
Verifying value of Plus30_Output_0
Verifying value of ReLU32_Output_0
Verifying value of Pooling66_Output_0
Verifying value of Convolution110_Output_0
Verifying value of Plus112_Output_0
Verifying value of ReLU114_Output_0
Verifying value of Pooling160_Output_0
Verifying value of Pooling160_Output_0_reshape0
Verifying value of Times212_Output_0
Verifying value of Plus214_Output_0
```

## ヘルプ情報

```bash
python3 debug.py -h
usage: debug.py [-h] model_path

positional arguments:
  model_path  Path to the model to debug.

optional arguments:
  -h, --help  show this help message and exit
```

