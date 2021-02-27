---
title: "ONNX-MLIRにmacOSのCIビルドを追加したときの試行錯誤メモ"
slug: "how-i-added-macos-ci-support-to-onnx-mlir"
subtitle:    ""
description: ""
date:        "2021-02-27"
author:      "Masahiro Hiramori"
image:       ""
tags:        ["ONNX", "MLIR", "GitHub Actions"]
categories:  ["Tech"]
draft:       true
---

## モチベーション

- macOSでONNX-MLIRをビルドできた
- CIビルドされてなかったので、CIの勉強もかねて追加してみた

## CIサービスの選定

- Travis CI
  - メンテナはこちらを推薦してきたが、macOSがサポートされてなくて断念
- GitHub Actions
  - こちらはmacOSをサポートしているため提案したところ、問題ないとのことで選択

## 設計

- Pythonバージョン
  - Python 3.7.8を使用
    - actions/pythonの制約で3.7.0が指定できないため
- ビルドスクリプト
  - 公式手順にしたがったビルド
  - 1回のビルドに1時間20分程度かかることが判明
  - キャッシュを活用することで高速化
- キャッシュ範囲の検討
  - キャッシュ対象は圧縮されて格納される
    - 圧縮後のキャッシュファイルが保存される上限は5GB
  - llvm-projectディレクトリ
    - キャッシュキーはclone-milr.shとbuild-mlir.sh
  - キャッシュサイズが上限に収まるか確認
    - →問題なし
## キャッシュキー衝突によりCIが失敗する問題

- 問題の内容
  - 同時に複数のビルドが走ると、キャッシュキーの衝突が起きる
    - キャッシュキーの確保→キャッシュキーに対してコンテンツを格納
- 解決策
  - ビルドキャッシュをあきらめる
  - キャッシュキーにバージョン番号を含めて、衝突が起きたらバージョンを上げる
    - →メンテナーに提案したところ、後者で行くこととなった

## キャッシュキー衝突によりCIが失敗する問題

- cache size
  - Release 
    - llvm-projectディレクトリをキャッシュ：1814 MB
    - llvm-project/buildディレクトリをキャッシュ：629 MB
  - MinSizeRel 
    - llvm-projectディレクトリをキャッシュ：1658 MB
    - llvm-project/buildディレクトリをキャッシュ：681 MB
- build time 
  - Release 
    - llvm-projectディレクトリをキャッシュ
      - キャッシュミス：1h 12m 0s
      - キャッシュヒット：12m 37s
    - llvm-project/buildディレクトリをキャッシュ 
      - キャッシュミス：1h 1m 29s
      - キャッシュヒット：14m 18s
  - MinSizeRel
    - llvm-projectディレクトリをキャッシュ 
      - キャッシュミス：1h 1m 26s
      - キャッシュヒット：13m 24s
    - llvm-project/buildディレクトリをキャッシュ 
      - キャッシュミス：1h 14m 34s
      - キャッシュヒット：18m 20s
