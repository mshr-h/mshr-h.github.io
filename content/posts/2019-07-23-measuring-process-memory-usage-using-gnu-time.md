---
title: "GNU timeでプロセスのメモリ使用量を取得する"
slug: "measuring-process-memory-usage-using-gnu-time"
subtitle:    ""
description: ""
date:        "2019-07-23"
author:      "Masahiro Hiramori"
tags:        []
categories:  ["Tech" ]
draft:       false
---

# TD;LR

```bash
/usr/bin/time -f "%M KB" command
```

# GNU time

GNU timeには、プロセスが使用したユーザ/システム時間だけでなく、最大メモリ使用量(Resident Set Size)を取得することができる。例えば、 `git --version`を実行したときの最大メモリ使用量を表示するには、次のコマンドを実行する。

```bash
$ /usr/bin/time -f "RSS: %M KB" git --version
git version 2.17.1
RSS: 1312 KB
```

`-f`オプションで出力フォーマットを指定し、引数に`%M`を指定することで最大メモリ使用量を表示する。

```
OPTIONS
       -f FORMAT, --format FORMAT
              Use FORMAT as the format string that controls the output of time.  See the below more information.
......
The resource specifiers, which are a superset of those recognized by the tcsh(1) builtin `time' command, are:
......
              M      Maximum resident set size of the process during its lifetime, in Kilobytes.
```