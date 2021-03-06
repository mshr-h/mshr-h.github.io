---
title: "pipdeptreeでPythonパッケージの依存ツリーを見る"
slug: "pipdeptree"
subtitle:    ""
description: ""
date:        "2020-09-21"
author:      "Masahiro Hiramori"
image:       ""
tags:        ["Python"]
categories:  ["Tech"]
draft:       false
---

とあるPythonパッケージが依存しているパッケージを表示するには、`pipdeptree`が便利。
オプションの指定なしで実行すると、全パッケージの依存リストが表示される。
`-p package_name`オプションで特定のパッケージに絞ることができる。

## tensorflowの依存一覧を表示する例

```bash
$ pipdeptree -p tensorflow
```
### 実行結果

```bash
tensorflow==2.1.0
  - absl-py [required: >=0.7.0, installed: 0.9.0]
    - six [required: Any, installed: 1.13.0]
  - astor [required: >=0.6.0, installed: 0.8.1]
  - gast [required: ==0.2.2, installed: 0.2.2]
  - google-pasta [required: >=0.1.6, installed: 0.1.8]
    - six [required: Any, installed: 1.13.0]
  - grpcio [required: >=1.8.6, installed: 1.26.0]
    - six [required: >=1.5.2, installed: 1.13.0]
  - keras-applications [required: >=1.0.8, installed: 1.0.8]
    - h5py [required: Any, installed: 2.10.0]
      - numpy [required: >=1.7, installed: 1.18.1]
      - six [required: Any, installed: 1.13.0]
    - numpy [required: >=1.9.1, installed: 1.18.1]
  - keras-preprocessing [required: >=1.1.0, installed: 1.1.0]
    - numpy [required: >=1.9.1, installed: 1.18.1]
    - six [required: >=1.9.0, installed: 1.13.0]
  - numpy [required: >=1.16.0,<2.0, installed: 1.18.1]
  - opt-einsum [required: >=2.3.2, installed: 3.1.0]
    - numpy [required: >=1.7, installed: 1.18.1]
  - protobuf [required: >=3.8.0, installed: 3.11.2]
    - setuptools [required: Any, installed: 41.2.0]
    - six [required: >=1.9, installed: 1.13.0]
  - scipy [required: ==1.4.1, installed: 1.4.1]
    - numpy [required: >=1.13.3, installed: 1.18.1]
  - six [required: >=1.12.0, installed: 1.13.0]
  - tensorboard [required: >=2.1.0,<2.2.0, installed: 2.1.0]
    - absl-py [required: >=0.4, installed: 0.9.0]
      - six [required: Any, installed: 1.13.0]
    - google-auth [required: >=1.6.3,<2, installed: 1.10.1]
      - cachetools [required: >=2.0.0,<5.0, installed: 4.0.0]
      - pyasn1-modules [required: >=0.2.1, installed: 0.2.8]
        - pyasn1 [required: >=0.4.6,<0.5.0, installed: 0.4.8]
      - rsa [required: >=3.1.4,<4.1, installed: 4.0]
        - pyasn1 [required: >=0.1.3, installed: 0.4.8]
      - setuptools [required: >=40.3.0, installed: 41.2.0]
      - six [required: >=1.9.0, installed: 1.13.0]
    - google-auth-oauthlib [required: >=0.4.1,<0.5, installed: 0.4.1]
      - google-auth [required: Any, installed: 1.10.1]
        - cachetools [required: >=2.0.0,<5.0, installed: 4.0.0]
        - pyasn1-modules [required: >=0.2.1, installed: 0.2.8]
          - pyasn1 [required: >=0.4.6,<0.5.0, installed: 0.4.8]
        - rsa [required: >=3.1.4,<4.1, installed: 4.0]
          - pyasn1 [required: >=0.1.3, installed: 0.4.8]
        - setuptools [required: >=40.3.0, installed: 41.2.0]
        - six [required: >=1.9.0, installed: 1.13.0]
      - requests-oauthlib [required: >=0.7.0, installed: 1.3.0]
        - oauthlib [required: >=3.0.0, installed: 3.1.0]
        - requests [required: >=2.0.0, installed: 2.22.0]
          - certifi [required: >=2017.4.17, installed: 2019.11.28]
          - chardet [required: >=3.0.2,<3.1.0, installed: 3.0.4]
          - idna [required: >=2.5,<2.9, installed: 2.8]
          - urllib3 [required: >=1.21.1,<1.26,!=1.25.1,!=1.25.0, installed: 1.25.7]
    - grpcio [required: >=1.24.3, installed: 1.26.0]
      - six [required: >=1.5.2, installed: 1.13.0]
    - markdown [required: >=2.6.8, installed: 3.1.1]
      - setuptools [required: >=36, installed: 41.2.0]
    - numpy [required: >=1.12.0, installed: 1.18.1]
    - protobuf [required: >=3.6.0, installed: 3.11.2]
      - setuptools [required: Any, installed: 41.2.0]
      - six [required: >=1.9, installed: 1.13.0]
    - requests [required: >=2.21.0,<3, installed: 2.22.0]
      - certifi [required: >=2017.4.17, installed: 2019.11.28]
      - chardet [required: >=3.0.2,<3.1.0, installed: 3.0.4]
      - idna [required: >=2.5,<2.9, installed: 2.8]
      - urllib3 [required: >=1.21.1,<1.26,!=1.25.1,!=1.25.0, installed: 1.25.7]
    - setuptools [required: >=41.0.0, installed: 41.2.0]
    - six [required: >=1.10.0, installed: 1.13.0]
    - werkzeug [required: >=0.11.15, installed: 0.16.0]
    - wheel [required: >=0.26, installed: 0.33.6]
  - tensorflow-estimator [required: >=2.1.0rc0,<2.2.0, installed: 2.1.0]
  - termcolor [required: >=1.1.0, installed: 1.1.0]
  - wheel [required: >=0.26, installed: 0.33.6]
  - wrapt [required: >=1.11.1, installed: 1.11.2]
```