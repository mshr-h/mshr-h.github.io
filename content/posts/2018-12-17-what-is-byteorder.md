---
title: "バイトオーダー(エンディアン)についてメモ"
slug: "what-is-byteorder"
subtitle:    ""
description: ""
date:        2018-12-17
author:      "Masahiro Hiramori"
tags:        ["Computer Architecture"]
categories:  ["Tech"]
draft:       false
---

TensorFlowのソースコードを読んでいたら、エンディアンチェックのコードが含まれていたので改めて調べた。
該当するTensorFlowのコードは、リトルエンディアンならgrpcioを依存パッケージに含めるという処理。

- [setup.py](https://github.com/tensorflow/tensorflow/blob/v1.12.0/tensorflow/tools/pip_package/setup.py#L63-L67)

```python
if sys.byteorder == 'little':
  # grpcio does not build correctly on big-endian machines due
  # to lack of BoringSSL support.
  # See <https://github.com/tensorflow/tensorflow/issues/17882>.
  REQUIRED_PACKAGES.append('grpcio >= 1.8.6')
```

---

## エンディアンとは

マルチバイトのデータをバイト単位に変換する方法で、ビッグエンディアン、リトルエンディアンがある。プロセッサによってエンディアンは異なり、AMD64だとリトルエンディアン、IBM S390Xだとビッグエンディアン、ARMはリトルエンディアンとビッグエンディアンを切替可能なバイエンディアンを採用している。
例えば、`0x89ABCDEF`で構成される4バイトデータを、メモリアドレス`0x0010`から`0x0013`の領域へ格納するときを考える。
ビッグエンディアンの場合、

```
0x0010: 89
0x0011: AB
0x0012: CD
0x0013: EF
```

のように、下位アドレスに最上位バイトが格納される。
リトルエンディアンの場合、

```
0x0010: EF
0x0011: CD
0x0012: AB
0x0013: 89
```

のように、上位アドレスに最上位バイトが格納される。

## なぜエンディアンチェックのコードがTensorFlowに含まれているのか

TensorFlowはAMD64やARMに加えて、IBM S390X向けにもビルド済みバイナリを提供している。しかし、TensorFlowが依存するRPCフレームワークのgrpcioは、ビッグエンディアン向けに正しくビルドできないというバグがある。ビッグエンディアンのマシン上にgrpcioが導入されるのを回避するために、リトルエンディアンのマシンなら依存ライブラリにgrpcioも含める、というコードが記載されている。
