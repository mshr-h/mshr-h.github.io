---
title:       "あるPythonパッケージが依存するライブラリ一覧をwhlファイルから探す"
subtitle:    ""
description: ""
date:        2018-12-16
author:      "Masahiro Hiramori"
tags:        ["Python"]
categories:  ["Tech" ]
draft:       false
---

TensorFlowをソースコードからビルドすると、PythonパッケージとしてTensorFlowのwhlファイルが作成される。このwhlファイルが依存するライブラリ一覧は、whlファイル内のMETADATAに記載されている。TensorFlow 1.12.0を対象に、実際にMETADATAファイルの中身を見てみた。

---

## 検証環境

- Ubuntu 16.04 x86_64
- Python 3.6.6

## 手順

適当なディレクトリを作成し、TensorFlowのwhlファイルをダウンロード、解凍。

```
$ mkdir workspace; cd workspace
$ pip3 download tensorflow==1.12.0 — no-deps
$ unzip tensorflow-1.12.0-cp36-cp36m-manylinux1_x86_64.whl
```

解凍すると、以下の2つのディレクトリができあがる。

```
tensorflow-1.12.0.data
tensorflow-1.12.0.dist-info
```

`tensorflow-1.12.0.dist-info/METADATA`の中身は以下のようになっている。

```
Metadata-Version: 2.1
Name: tensorflow
Version: 1.12.0
Summary: TensorFlow is an open source machine learning framework for everyone.
Home-page: <https://www.tensorflow.org/>
Author: Google Inc.
Author-email: opensource@google.com
License: Apache 2.0
Download-URL: <https://github.com/tensorflow/tensorflow/tags>
Keywords: tensorflow tensor machine learning
Platform: UNKNOWN
Classifier: Development Status :: 5 — Production/Stable
Classifier: Intended Audience :: Developers
Classifier: Intended Audience :: Education
Classifier: Intended Audience :: Science/Research
Classifier: License :: OSI Approved :: Apache Software License
Classifier: Programming Language :: Python :: 2
Classifier: Programming Language :: Python :: 2.7
Classifier: Programming Language :: Python :: 3
Classifier: Programming Language :: Python :: 3.4
Classifier: Programming Language :: Python :: 3.5
Classifier: Programming Language :: Python :: 3.6
Classifier: Topic :: Scientific/Engineering
Classifier: Topic :: Scientific/Engineering :: Mathematics
Classifier: Topic :: Scientific/Engineering :: Artificial Intelligence
Classifier: Topic :: Software Development
Classifier: Topic :: Software Development :: Libraries
Classifier: Topic :: Software Development :: Libraries :: Python Modules
Requires-Dist: absl-py (>=0.1.6)
Requires-Dist: astor (>=0.6.0)
Requires-Dist: gast (>=0.2.0)
Requires-Dist: keras-applications (>=1.0.6)
Requires-Dist: keras-preprocessing (>=1.0.5)
Requires-Dist: numpy (>=1.13.3)
Requires-Dist: six (>=1.10.0)
Requires-Dist: protobuf (>=3.6.1)
Requires-Dist: tensorboard (<1.13.0,>=1.12.0)
Requires-Dist: termcolor (>=1.1.0)
Requires-Dist: grpcio (>=1.8.6)
Requires-Dist: wheel (>=0.26)
TensorFlow is an open source software library for high performance numerical
computation. Its flexible architecture allows easy deployment of computation
across a variety of platforms (CPUs, GPUs, TPUs), and from desktops to clusters
of servers to mobile and edge devices.
Originally developed by researchers and engineers from the Google Brain team
within Google’s AI organization, it comes with strong support for machine
learning and deep learning and the flexible numerical computation core is used
across many other scientific domains.
```

`Requires-Dist:`で始まる行が、Pythonパッケージが依存するライブラリを表している。TensorFlow 1.12.0の場合、以下のライブラリに依存していることが分かる。

```
absl-py (>=0.1.6)
astor (>=0.6.0)
gast (>=0.2.0)
keras-applications (>=1.0.6)
keras-preprocessing (>=1.0.5)
numpy (>=1.13.3)
six (>=1.10.0)
protobuf (>=3.6.1)
tensorboard (<1.13.0,>=1.12.0)
termcolor (>=1.1.0)
grpcio (>=1.8.6)
wheel (>=0.26)
```

以上で、TensorFlow 1.12.0のwhlファイルから、依存するライブラリ一覧を探すことができた。
