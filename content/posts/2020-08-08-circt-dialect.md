---
title: "CIRCTのDialect調査"
slug: "circt-dialect"
subtitle:    ""
description: ""
date:        "2020-08-08"
author:      "Masahiro Hiramori"
image:       ""
tags:        ["CIRCT", "MLIR"]
categories:  ["Tech"]
draft:       false
---

CIRCTに含まれるDialectとその変換パスを調査。
Dialectの変換はソースコードの以下に存在。
- [lib/Conversion](https://github.com/llvm/circt/tree/master/lib/Conversion)

## Dialect一覧

CIRCTには以下のDialectが含まれている。
- FIRRTL Dialect
- RTL Dialect
- Handshake Dialect
- StaticLogic Dialect
- LLHD Dialect

その他、MLIRでデフォルトで定義されているDialectとして以下がある。
- Standard Dialect
- LLVM Dialect

## Dialect変換パス

{{< figure src="/img/post/2020-08-08-circt-conversion-diagram.png" width="100%" height="100%"
    link="/img/post/2020-08-08-circt-conversion-diagram.png">}}

FIRRTLからVerilogを生成する流れは、
- FIR parserでFIRRTL Dialectへ変換し、
- FIRRTL DialectからRTL Dialectへ変換、
- 最後にRTL DialectからVerilogコードを生成する

となっている。
