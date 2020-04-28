---
title: "ONNX MLIRで出力したshared libraryを実行する"
slug: "run-onnx-mlir-shared-library"
subtitle:    ""
description: ""
date:        "2020-08-06"
author:      "Masahiro Hiramori"
image:       ""
tags:        ["ONNX", "MLIR"]
categories:  ["Tech"]
draft:       false
---

ONNX MLIRに付属の[debug.py](https://github.com/onnx/onnx-mlir/blob/master/utils/debug.py)をベースに作成。
ONNX MLIRでONNXモデルファイルをshared libraryに変換し、PyRuntimeで実行する。

以下はメモ。
- `ExecutionSession(shared_lib_path, "_dyn_entry_point_main_graph")`は、第2引数にエントリポイントを指定している。
- `ExecutionSession.run`でグラフを実行するときは、引数を辞書で与えなくていい
  - ONNX Runtimeだと`{name: value}`の辞書形式で与える必要がある

```python
import os
import sys
import argparse
import onnx
import subprocess
import numpy as np
import tempfile

from collections import OrderedDict

ONNX_MLIR = os.path.join(os.environ['ONNX_MLIR_HOME'], "bin/onnx-mlir")
RUNTIME_DIR = os.path.join(os.environ['ONNX_MLIR_HOME'], "lib")
sys.path.append(RUNTIME_DIR)

from PyRuntime import ExecutionSession

def main(model_path):
    model = onnx.load(model_path)
    subprocess.run([ONNX_MLIR, model_path], stdout=subprocess.PIPE, check=True)
    shared_lib_path = os.path.splitext(os.path.basename(model_path))[0] + ".so"
    sess = ExecutionSession(shared_lib_path, "_dyn_entry_point_main_graph")

    # calculate input shape
    shape_proto = model.graph.input[0].type.tensor_type.shape
    explicit_shape = []
    for dim in shape_proto.dim:
        explicit_shape.append(dim.dim_value)

    # generate input value
    np.random.seed(42)
    inputs = np.random.uniform(-1.0, 1.0, explicit_shape).astype(np.float32)

    outs = sess.run(inputs)
    print(outs)

if __name__ == '__main__':
    parser = argparse.ArgumentParser()
    parser.add_argument('model_path', type=str, help="Path to the model to debug.")
    args = parser.parse_args()
    main(**vars(args))
```
