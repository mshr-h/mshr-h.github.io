---
title: "MLIRの標準Dialect調査"
slug: "mlir-default-dialect"
subtitle:    ""
description: ""
date:        "2020-08-08"
author:      "Masahiro Hiramori"
image:       ""
tags:        ["MLIR"]
categories:  ["Tech"]
draft:       false
---

MLIRでデフォルトで定義されているDialectとその変換パスを調査。
Dialectの変換はソースコードの以下に存在。
- [mlir/lib/Conversion](https://github.com/llvm/llvm-project/tree/master/mlir/lib/Conversion)

## Dialect一覧

MLIRには以下のDialectが含まれている。
- Affine Dialect
- AVX512 Dialect
- GPU Dialect
- Linalg Dialect
- LLVM Dialect
- NVVM Dialect
- ROCDL Dialect
- SCF Dialect
- Shape Dialect
- SPIRV Dialect
- Standard Dialect
- Vector Dialect
- Vulkan Dialect

## Dialect変換パス

{{< figure src="/img/post/2020-08-08-mlir-default-dialect-conversion-diagram.png" width="100%" height="100%"
    link="/img/post/2020-08-08-mlir-default-dialect-conversion-diagram.png">}}

