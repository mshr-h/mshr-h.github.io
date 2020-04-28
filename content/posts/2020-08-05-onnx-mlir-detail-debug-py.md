---
title: "ONNX MLIRのdebug.py詳細"
slug: "onnx-mlir-detail-debug-py"
subtitle:    ""
description: ""
date:        "2020-08-05"
author:      "Masahiro Hiramori"
image:       ""
tags:        ["ONNX", "MLIR", "Python"]
categories:  ["Tech"]
draft:       false
---

[ONNX MLIRに付属のdebug.pyを動かす]({{< ref "/posts/2020-08-04-onnx-mlir-debug-py.md" >}})で動かした`debug.py`の中を見る。

## `debug.py`の処理内容

### 概要

- 入力として指定したONNXモデルを、ONNX MLIRでshared libraryとしてビルド、実行し、リファレンスバックエンドで実行した結果と比較する
- リファレンスバックエンドにはONNX Runtimeを使用
- Operatorのoutputごとに比較
  - モデルの出力だけでなく、Operatorの実行結果単位で比較している
- `PyRuntime`はおそらく、ONNX MLIRでビルドしたshared libraryを、Pythonから実行するためのPythonバインディング
  - shared libraryの実行方法は`PyRuntime`の実装を見る必要がありそう

### 詳細

`import onnxruntime`でONNX Runtimeをインポートする。

```python
import os
import sys
import argparse
import onnx
import subprocess
import numpy as np
import tempfile

from collections import OrderedDict

# Reference backend, use onnxruntime by default
import onnxruntime
prepare = onnxruntime.InferenceSession
```

`ONNX_MLIR_HOME`が設定されているか確認。

```python
if (not os.environ.get('ONNX_MLIR_HOME', None)):
    raise RuntimeError(
        "Environment variable ONNX_MLIR_HOME is not set, please set it to the path to "
        "the HOME directory for onnx-mlir. The HOME directory for onnx-mlir refers to "
        "the parent folder containing the bin, lib, etc sub-folders in which ONNX-MLIR "
        "executables and libraries can be found.")
```

`onnx-mlir`実行ファイルパス、`lib`ディレクトリをimport検索パスに追加などする。

```python
VERBOSE = os.environ.get('VERBOSE', False)
ONNX_MLIR = os.path.join(os.environ['ONNX_MLIR_HOME'], "bin/onnx-mlir")

# Include runtime directory in python paths, so PyRuntime can be imported.
RUNTIME_DIR = os.path.join(os.environ['ONNX_MLIR_HOME'], "lib")
sys.path.append(RUNTIME_DIR)
```

`PyRuntime`(ONNX MLIRでビルドしたshared libraryをPythonから実行するためのPythonバインディング？)をインポートする。

```python
try:
    from PyRuntime import ExecutionSession
except ImportError:
    raise ImportError(
        "Looks like you did not build the PyRuntime target, build it by running `make PyRuntime`."
    )
```

ユーティリティ関数を定義。

`extend_model_output`関数は、モデル内の各Operatorのoutputに、data type、shape情報を追加する。

```python
def execute_commands(cmds):
    if (VERBOSE):
        print(" ".join(cmds))
    subprocess.run(cmds, stdout=subprocess.PIPE, check=True)


def extend_model_output(model, intermediate_outputs):
    # onnx-mlir doesn't care about manually specified output types & shapes.
    DUMMY_TENSOR_TYPE = onnx.TensorProto.FLOAT

    while (len(model.graph.output)):
        model.graph.output.pop()

    for output_name in intermediate_outputs:
        output_value_info = onnx.helper.make_tensor_value_info(
            output_name, DUMMY_TENSOR_TYPE, None)
        model.graph.output.extend([output_value_info])
    return model
```

メイン関数。

```python
def main(model_path):
```

入力のONNXファイルを読み込み、各Operatorのoutputにdata type、shape情報を追加する。

```python
    model = onnx.load(model_path)
    intermediate_outputs = sum(
        [list(node.output) for node in model.graph.node], [])
    intermediate_outputs = list(OrderedDict.fromkeys(intermediate_outputs))
    model = extend_model_output(model, intermediate_outputs)
```

以下は一時ディレクトリ内で実行。

```python
    with tempfile.TemporaryDirectory() as temp_dir:
        print("Temporary directory has been created at {}".format(temp_dir))
```

ONNXモデルをファイルに出力し、ONNX MLIRでshared libraryに変換する。

```python
        # Save modified model & invoke onnx-mlir to compile it.
        temp_model_path = os.path.join(temp_dir, "model.onnx")
        onnx.save(model, temp_model_path)
        execute_commands([ONNX_MLIR, temp_model_path])
```

shared libraryから実行セッションを作成する。

```python
        # Use the generated shared library to create an execution session.
        temp_shared_lib_path = os.path.join(temp_dir, "model.so")
        sess = ExecutionSession(temp_shared_lib_path,
                                "_dyn_entry_point_main_graph")
```

入力データとしてランダムデータを生成する。

```python
        # Generate random data as input.
        inputs = []
        input_names = []
        initializers = list(map(lambda x: x.name, model.graph.initializer))
        np.random.seed(42)
        for input_proto in model.graph.input:
            if input_proto.name not in initializers:
                input_names.append(input_proto.name)
                shape_proto = input_proto.type.tensor_type.shape
                explicit_shape = []
                for dim in shape_proto.dim:
                    assert dim.dim_value, "Can only debug models with inputs that have explicit shapes."
                    explicit_shape.append(dim.dim_value)
                inputs.append(
                    np.random.uniform(-1.0, 1.0, explicit_shape).astype(np.float32))
```

shared libraryの実行セッションを実行する。

```python
        # Run the compiled inference function on the randomly generated data.
        outs = sess.run(inputs)
```

リファレンスバックエンド(ONNX Runtime)で実行する。

```python
        # Run the model with reference backend and get results.
        ref_session = prepare(temp_model_path)
        output_names = list(map(lambda x: x.name, model.graph.output))
        input_feed = dict(zip(input_names, inputs))
        ref_outs = ref_session.run(output_names, input_feed)
```

shared libraryとONNX Runtimeで実行した結果を比較する。

```python
        # For each intermediate output tensor, compare results.
        for i, name in enumerate(intermediate_outputs):
            print("Verifying value of {}".format(name))
            np.testing.assert_array_almost_equal(ref_outs[i], outs[i], decimal=5)
```

引数パーサーを定義し、メイン関数へパース結果を渡す。

```python
if __name__ == '__main__':
    parser = argparse.ArgumentParser()
    parser.add_argument('model_path', type=str, help="Path to the model to debug.")
    args = parser.parse_args()
    main(**vars(args))
```
