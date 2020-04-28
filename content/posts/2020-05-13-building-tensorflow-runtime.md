---
title: "TFRT(TensorFlow Runtime)をUbuntu 18.04 on WSLでビルドした"
slug: "building-tensorflow-runtime"
subtitle:    ""
description: ""
date:        "2020-05-13"
author:      "Masahiro Hiramori"
image:       ""
tags:        ["TensorFlow", "Linux"]
categories:  ["Tech" ]
draft:       false
---

TFRT(TensorFlow Runtime)は、TensorFlowの新しい実行ランタイムでパフォーマンスが良いらしい。
Linux(WSL)上でビルドしたので、メモ。

# 実施環境

- Ryzen 5 1600
- 32GB RAM
- Ubuntu 18.04 on WSL1

# 必要なツール導入

`Clang 9`以上と`libstdc++ 8`以上、`Bazel 2.0.0〜3.1.0`が必要なので、導入する。

```bash
$ sudo apt update
$ sudo apt install clang-9
$ sudo update-alternatives --install /usr/bin/clang clang /usr/bin/clang-9 9
$ sudo update-alternatives --install /usr/bin/clang++ clang++ /usr/bin/clang++-9 9
$ sudo add-apt-repository -y ppa:ubuntu-toolchain-r/test
$ sudo apt-get update
$ sudo apt-get install -y gcc-8 g++-8
$ wget https://github.com/bazelbuild/bazel/releases/download/3.1.0/bazel-3.1.0-linux-x86_64
$ mv ./bazel-3.1.0-linux-x86_64 ~/bin/bazel # PATHが通っているディレクトリ
```

ちゃんと導入できたか確認。

```bash
$ clang++ -v |& grep "Selected GCC"
Selected GCC installation: /usr/bin/../lib/gcc/x86_64-linux-gnu/8
$ bazel --version
bazel 3.1.0
```

# ビルド実施

ソースコードを取得し、ビルド開始。

```bash
$ cd ~/workspace
$ git clone https://github.com/tensorflow/runtime
$ cd runtime
$ bazel build -c opt //tools:bef_executor
$ bazel build -c opt //tools:tfrt_translate
```

`bef_executor`のビルドに33分、`tfrt_translate`のビルドに2.5分ぐらいかかった。TensorFlow本体のビルドは1.5時間程度かかったので、それを比べるとかなり短い。

# テスト実行

用意されているテストをいくつか実行してみる。

```
$ bazel-bin/tools/tfrt_translate -mlir-to-bef mlir_tests/bef_executor/async.mlir | bazel-bin/tools/bef_executor
Choosing memory leak check allocator.
Choosing single-threaded work queue.
--- Not running '' because it has arguments.
--- Running 'async_add':
int32 = 42
int32 = 42
int32 = 43
--- Running 'async_repeat2':
int32 = -1
hello host executor!
int32 = 0
hello host executor!
int32 = 0
--- Not running 'call_async_add.i32' because it has arguments.
--- Running 'async_add.i32_caller':
int32 = 42
int32 = 42
int32 = 44
--- Running 'test_async_print_result':
'test_async_print_result' returned 43
--- Running 'test_async_copy':
'test_async_copy' returned 42
--- Running 'test_async_copy.with_delay':
'test_async_copy.with_delay' returned 42
--- Running 'test_async_copy_2':
'test_async_copy_2' returned 43
```

```
$ bazel test //integrationtest/mnist:mnist.mlir.test
INFO: Analyzed target //integrationtest/mnist:mnist.mlir.test (6 packages loaded, 215 targets configured).
INFO: Found 1 test target...
Target //integrationtest/mnist:mnist.mlir.test up-to-date:
  bazel-bin/integrationtest/mnist/mnist.mlir.test
INFO: Elapsed time: 209.614s, Critical Path: 21.09s
INFO: 8 processes: 8 processwrapper-sandbox.
INFO: Build completed successfully, 15 total actions
//integrationtest/mnist:mnist.mlir.test                                  PASSED in 10.8s

Executed 1 out of 1 test: 1 test passes.
There were tests whose specified size is too big. Use the --test_verbose_timeout_warnings command line option INFO: Build completed successfully, 15 total actions
```

```
$ bazel test //integrationtest/mnist:mnist_training.mlir.test
INFO: Analyzed target //integrationtest/mnist:mnist_training.mlir.test (0 packages loaded, 2 targets configured).
INFO: Found 1 test target...
Target //integrationtest/mnist:mnist_training.mlir.test up-to-date:
  bazel-bin/integrationtest/mnist/mnist_training.mlir.test
INFO: Elapsed time: 2.932s, Critical Path: 0.88s
INFO: 2 processes: 2 processwrapper-sandbox.
INFO: Build completed successfully, 5 total actions
//integrationtest/mnist:mnist_training.mlir.test                         PASSED in 0.5s

Executed 1 out of 1 test: 1 test passes.
There were tests whose specified size is too big. Use the --test_verbose_timeout_warnings command line option INFO: Build completed successfully, 5 total actions
```

テストが通っており、正しくビルドできたみたい。
