---
title: "Linux(WSL)上でTensorFlowをソースコードからビルドする"
slug: "how-to-build-from-tensorflow-on-wsl"
subtitle:    ""
description: ""
date:        2018-12-19
author:      "Masahiro Hiramori"
tags:        []
categories:  ["Tech"]
draft:       false
---

ディープラーニングフレームワークのTensorFlowは、Googleが開発しており、ビルド済みのTensorFlowも提供されている。しかし、動作させるCPUによって、対応する拡張命令は異なる。ソースコードからビルドすることで、特定の環境でTensorFlowを高速に動作することが可能となる。
今回は、Linux(WSL)上でTensorFlowをソースコードからビルドする方法について書く。ビルドに使用するPC環境は以下のとおり。

- Ryzen 5 1600
- 32GBメモリ
- Windows Subsystem for Linux(Ubuntu 18.04)

今回は以下の構成でビルドする。

- TensorFlow v1.12.0
- Python 2
- GPUサポートは無し

基本的には、以下の公式ページ通りに進めることでビルドできるが、最新版Bazelを使うとビルドエラーになるため、古いバージョンを導入する必要があることに注意。

- [Build from source | TensorFlow](https://www.tensorflow.org/install/source)

ビルドの流れとしては、

- 必要なPythonパッケージ、ビルドツールの導入
- ソースコードの取得
- ビルドコンフィグの生成
- ビルド実施

の順で進める。

## ビルドに必要なPythonパッケージの導入

公式手順通りにPythonパッケージを導入する。

```bash
$ sudo apt install -y python-dev python-pip
$ pip install -U --user pip six numpy wheel mock
$ pip install -U --user keras_applications==1.0.6 --no-deps
$ pip install -U --user keras_preprocessing==1.0.5 --no-deps
```

## Bazelの導入

2018/12/16時点の最新版(0.20.0)だとビルドできないので、古い0.15.0を導入する。

```bash
$ wget https://github.com/bazelbuild/bazel/releases/download/0.15.0/bazel_0.15.0-linux-x86_64.deb
$ sudo dpkg -i bazel_0.15.0-linux-x86_64.deb
$ sudo apt install -f -y
```

## TensorFlowソースコード取得

TensorFlowのソースコードを取得します。今回は、v1.12.0をビルドするので、該当tagをチェックアウトする。

```bash
$ git clone https://github.com/tensorflow/tensorflow
$ cd tensorflow
$ git checkout v1.12.0
```

## ビルドコンフィグ生成

ビルドコンフィグを生成する。今回はデフォルトのままで進める。

```bash
$ ./configure
WARNING: The following rc files are no longer being read, please transfer their contents or import their path into one of the standard rc files:
/home/mshr/tensorflow/tools/bazel.rc
WARNING: --batch mode is deprecated. Please instead explicitly shut down your Bazel server using the command "bazel shutdown".
INFO: Invocation ID: 2e09c61f-cc51-4ae2-9704-46e8b8f239f5
You have bazel 0.20.0 installed.
Please specify the location of python. [Default is /usr/bin/python]:
Found possible Python library paths:
  /usr/local/lib/python2.7/dist-packages
  /usr/lib/python2.7/dist-packages
Please input the desired Python library path to use.  Default is [/usr/local/lib/python2.7/dist-packages]
Do you wish to build TensorFlow with Apache Ignite support? [Y/n]:
Apache Ignite support will be enabled for TensorFlow.
Do you wish to build TensorFlow with XLA JIT support? [Y/n]:
XLA JIT support will be enabled for TensorFlow.
Do you wish to build TensorFlow with OpenCL SYCL support? [y/N]:
No OpenCL SYCL support will be enabled for TensorFlow.
Do you wish to build TensorFlow with ROCm support? [y/N]:
No ROCm support will be enabled for TensorFlow.
Do you wish to build TensorFlow with CUDA support? [y/N]:
No CUDA support will be enabled for TensorFlow.
Do you wish to download a fresh release of clang? (Experimental) [y/N]:
Clang will not be downloaded.
Do you wish to build TensorFlow with MPI support? [y/N]:
No MPI support will be enabled for TensorFlow.
Please specify optimization flags to use during compilation when bazel option "--config=opt" is specified [Default is -march=native]:
Would you like to interactively configure ./WORKSPACE for Android builds? [y/N]:
Not configuring the WORKSPACE for Android builds.
Preconfigured Bazel build configs. You can use any of the below by adding "--config=<>" to your build command. See tools/bazel.rc for more details.
        --config=mkl            # Build with MKL support.
        --config=monolithic     # Config for mostly static monolithic build.
        --config=gdr            # Build with GDR support.
        --config=verbs          # Build with libverbs support.
        --config=ngraph         # Build with Intel nGraph support.
Configuration finished
```

## ビルド実施

ビルドを開始する。今回のビルド環境だと1時間半程度かかる。

```
$ bazel build --config=opt //tensorflow/tools/pip_package:build_pip_package
・・省略・・
INFO: From Executing genrule //tensorflow/python/estimator/api:estimator_python_api_gen:
tf.estimator package not installed.
tf.estimator package not installed.
Target //tensorflow/tools/pip_package:build_pip_package up-to-date:
  bazel-bin/tensorflow/tools/pip_package/build_pip_package
INFO: Elapsed time: 5474.748s, Critical Path: 172.25s
INFO: 10641 processes: 10641 local.
INFO: Build completed successfully, 11347 total actions
ビルド完了後、PythonのPIPパッケージを作成します。作成には15分ほどかかります。
$ ./bazel-bin/tensorflow/tools/pip_package/build_pip_package /tmp/tensorflow_pkg
Sat Dec 15 22:47:56 DST 2018 : === Preparing sources in dir: /tmp/tmp.qmE6tTngOh
~/tensorflow ~/tensorflow
~/tensorflow
Sat Dec 15 22:49:18 DST 2018 : === Building wheel
warning: no files found matching '*.pd' under directory '*'
warning: no files found matching '*.dll' under directory '*'
warning: no files found matching '*.lib' under directory '*'
warning: no files found matching '*.h' under directory 'tensorflow/include/tensorflow'
warning: no files found matching '*' under directory 'tensorflow/include/Eigen'
warning: no files found matching '*.h' under directory 'tensorflow/include/google'
warning: no files found matching '*' under directory 'tensorflow/include/third_party'
warning: no files found matching '*' under directory 'tensorflow/include/unsupported'
Sat Dec 15 22:50:10 DST 2018 : === Output wheel file is in: /tmp/tensorflow_pkg
```

## ビルドしたTensorFlowの導入

pipコマンドによりビルドしたTensorFlowを導入する。

```
$ pip install --user /tmp/tensorflow_pkg/tensorflow-1.12.0-cp27-cp27mu-linux_x86_64.whl
```

## MNISTを学習

ビルドしたTensorFlowの動作確認のために、TensorFlow公式チュートリアルに記載のMNISTを学習する。

```python
import tensorflow as tf
mnist = tf.keras.datasets.mnist

(x_train, y_train),(x_test, y_test) = mnist.load_data()
x_train, x_test = x_train / 255.0, x_test / 255.0

model = tf.keras.models.Sequential([
  tf.keras.layers.Flatten(),
  tf.keras.layers.Dense(512, activation=tf.nn.relu),
  tf.keras.layers.Dropout(0.2),
  tf.keras.layers.Dense(10, activation=tf.nn.softmax)
])
model.compile(optimizer='adam',
              loss='sparse_categorical_crossentropy',
              metrics=['accuracy'])

model.fit(x_train, y_train, epochs=5)
model.evaluate(x_test, y_test)
```

以上で、TensorFlowをソースコードからビルドし、MNISTの学習を実施し、ビルドしたTensorFlowの動作を確認した。次回は、拡張命令の有無による動作速度の変化を検証したい。