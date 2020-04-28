---
title: "Jupyter Notebook上に現在のノートブックのメモリ使用量を表示する"
slug: "display-memory-usage-on-jupyter-notebook"
subtitle:    ""
description: ""
date:        "2019-03-10"
author:      "Masahiro Hiramori"
tags:        []
categories:  ["Tech" ]
draft:       false
---

Jupyter Notebook上でPythonコードを実行中に、メモリ使用量を確認したいときがある。次のプラグインをインストールすると、Jupyter Notebook上にメモリ使用量を表示できる。

- [yuvipanda/nbresuse](https://github.com/yuvipanda/nbresuse)

インストール方法は次のコマンドを実行する。

```
pip install nbresuse
```

Jupyter Notebookを起動すると、下図赤枠のようにメモリ使用量が表示さる。

![/img/post/2019-03-10-jupyter-notebook.png](/img/post/2019-03-10-jupyter-notebook.png)