---
title: "C/C++デバッグプリントプラクティス"
slug: "c-cpp-debug-print-practice"
subtitle:    ""
description: ""
date:        "2020-02-22"
author:      "Masahiro Hiramori"
tags:        []
categories:  ["Tech" ]
draft:       false
---

# TD;LR

```c
#include <stdio.h>

printf("\\t%s:%dL %s() : some_variable=%d\\n",
  __FILE__, __LINE__, __func__, some_variable);
```

# はじめに

そこそこ大規模なOSS([TensorFlow](https://github.com/tensorflow/tensorflow)というディープラーニングフレームワーク)の動作解析に役立ったコードスニペットを紹介。特に、関数間の呼び出し順序を解析するときに役に立った。

# 説明

printf関数でソースコードのファイルパス、printf呼び出しの行数、printf呼び出し元の関数名を出力する。必要に応じて関数内に定義された変数も出力する。このコードスニペットで使っている事前定義識別子の説明は以下のとおり。

- `__FILE__`
    - ソースコードのファイル名がフルパスで文字列として定義されている変数
- `__LINE__`
    - ソースコード中の行数が数値として定義されている変数
- `__func__`
    - 関数名が文字列として定義されている変数

# 使い方

呼び出し順序を解析したい関数の先頭に上記のコードスニペットを貼り付けて、ビルド、実行するだけ。