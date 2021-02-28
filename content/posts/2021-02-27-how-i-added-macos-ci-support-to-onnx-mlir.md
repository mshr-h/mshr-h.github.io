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

2020/10に[ONNX-MLIR](https://github.com/onnx/onnx-mlir)という機械学習コンパイラ(学習済みモデルを実行可能バイナリに変換するもの)に
[macOSのCI設定を追加するPR](https://github.com/onnx/onnx-mlir/pull/287)を送り、マージされました。
本記事では、このPRを作成する上でどういう試行錯誤をしたかを書きます。

## 経緯

OSS機械学習コンパイラの調査の一環でONNX-MLIRを見つけ、開発用PC(macOS; MacBook Pro 13")に導入していました。
リポジトリ内を見ていたところ、x86-Linux、s390-Linux、x86-WindowsのCIは走っていますが、どうやらmacOSのCIは無いみたいでした。
Issueに「macOS向けCIパイプラインを追加しようと思ってるけどどう？」と[聞いてみた](https://github.com/onnx/onnx-mlir/issues/252)ところ、
メンテナの一人から是非やってやって欲しい旨のコメントがあり、作業に着手しました。

## CIサービスの選定

GitHubをサポートしているCIサービスはいくつかありますが、メンテナからはTravis CIでやって欲しいとの要望があったため、
Travis CI向けに調査、検討を始めました。

ONNX-MLIRは、ビルドにDockerを使っています。
単一のDockerイメージでビルドするのではなく、以下のようにビルドを2段階に分ける構成にすることで、
依存ライブラリのビルド成果物を再利用可能となり、ビルド時間の短縮を実現しています。

1. 依存ライブラリをビルドするDockerイメージ
   - `llvm/mlir`が一番大きな依存ライブラリで、ビルドに1時間程度かかる
2. 上記でビルドした生成物を使ってONNX-MLIR本体をビルドするDockerイメージ
   - このイメージはTravis-CI実行時に毎回ビルドされる

Travis CIとDockerのmacOSサポート状況を調査したところ、

- DockerはゲストOSとしてのmacOSをサポートしていない(多分)
- Travis CIのmacOS環境はDockerをサポートしていない([Using Docker in Builds](https://docs.travis-ci.com/user/docker/))

ため、Travis CIは使いないと判断しました。

ただ、実現したいことは「キャッシュ機構による高速化」であることは分かった、代替案の調査に着手しました。

別のCIサービスとしてGitHub Actionsを調査したところ、[キャッシュ機構](https://docs.github.com/en/actions/guides/caching-dependencies-to-speed-up-workflows)をサポートしており、これを提案しようと考えました。
しかし、GitHub Actionsのキャッシュ機構は以下の制約があることがわかりました。

1. 7日間以上アクセスのないキャッシュエントリは削除される
1. リポジトリ全体でキャッシュサイズの上限が5GB

上記の制約があるが、GitHub Actionsは使えそうとのことをメンテナに相談したところ、
`1.`の制約は問題なく、`2.`が大丈夫ならGitHub Actionsを使おうとのコメントをもらいました。

# メモ

- Travis CI
  - メンテナはこちらを推薦してきたが、macOSがサポートされてなくて断念
- GitHub Actions
  - こちらはmacOSをサポートしているため提案したところ、問題ないとのことで選択

(今考えれば、Travis CIのキャッシュ機構を使えば同じことができますね)

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
