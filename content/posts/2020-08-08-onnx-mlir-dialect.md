---
title: "ONNX MLIRのDialect調査"
slug: "onnx-mlir-dialect"
subtitle:    ""
description: ""
date:        "2020-08-08"
author:      "Masahiro Hiramori"
image:       ""
tags:        ["ONNX", "MLIR"]
categories:  ["Tech"]
draft:       false
---

ONNX MLIRに含まれるDialectとその変換パスを調査。
Dialectの変換はソースコードの以下に存在。
- [src/Conversion](https://github.com/onnx/onnx-mlir/tree/master/src/Conversion)

## Dialect一覧

ONNX MLIRには以下のDialectが含まれている。
- ONNX Dialect
- Krnl Dialect
- Handshake Dialect
- StaticLogic Dialect
- LLHD Dialect

その他、MLIRでデフォルトで定義されているDialectとして以下がある。
- Affine Dialect
- Standard Dialect
- LLVM Dialect

## Dialect変換パス

{{< figure src="/img/post/2020-08-08-onnx-mlir-conversion-diagram.png" width="100%" height="100%"
    link="/img/post/2020-08-08-onnx-mlir-conversion-diagram.png">}}

ONNXモデルをLLVM IRを生成する流れは、
- ONNX parserでONNXモデルからONNX Dialectへ変換し、
- ONNX DialectからKrnl Dialectへ変換、
- Krnl DialectからLLVM Dialectへ変換し、
- 最後にLLVM DialectからLLVM IRを生成する
