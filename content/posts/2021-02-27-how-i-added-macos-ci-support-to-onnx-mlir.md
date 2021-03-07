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

別のCIサービスとしてGitHub Actionsを調査したところ、[キャッシュ機構](https://docs.github.com/en/actions/guides/caching-dependencies-to-speed-up-workflows)(`actions/cache@v2`)をサポートしており、これを提案しようと考えました。

(今考えると、Travis CIのキャッシュ機構を使えば同じことができますね。。。)

しかし、GitHub Actionsのキャッシュ機構は以下の制約があることがわかりました。

1. 7日間以上アクセスのないキャッシュエントリは削除される
2. リポジトリ全体でキャッシュサイズの上限が5GB

上記の制約があるが、GitHub Actionsは使えそうとのことをメンテナに相談したところ、
`1.`の制約は問題なく、`2.`が大丈夫ならGitHub Actionsを使おうとのコメントをもらいました。
というわけで、キャッシュ対象のファイルサイズの調査をします。

## `actions/cache@v2`調査

キャッシュ対象のファイルサイズの調査の前に、どのようにキャッシュされているか知るために`actions/cache@v2`を調査します。
GitHub Actionsの`actions/cache@v2`は、リポジトリ全体でキャッシュサイズの上限が5GBという制約があります。
実はこのサイズは、キャッシュ対象の圧縮後のサイズを指しています。(公式ドキュメントには記載がありません)

以下のようにコードを読んでいくと、キャッシュ対象は`Gzip` or `zstd`で圧縮されることがわかります。

- キャッシュ保存時に呼ばれるメソッドは`saveCache()`を呼び出しています。
  - [https://github.com/actions/cache/blob/releases/v2/src/save.ts#L40](https://github.com/actions/cache/blob/releases/v2/src/save.ts#L40)

```typescript
await cache.saveCache(cachePaths, primaryKey);
```

- 中では`createTar()`を呼び出しています。
  - [https://github.com/actions/toolkit/blob/main/packages/cache/src/cache.ts#L136-L189](https://github.com/actions/toolkit/blob/main/packages/cache/src/cache.ts#L136-L189)

```typescript
const compressionMethod = await utils.getCompressionMethod()

・・・省略・・・

await createTar(archiveFolder, cachePaths, compressionMethod)
```

- `createTar()`呼び出し時の引数`compressionMethod`は、`getCompressionMethod()`で決めています。
  - [https://github.com/actions/toolkit/blob/main/packages/cache/src/internal/cacheUtils.ts#L86-L106](https://github.com/actions/toolkit/blob/main/packages/cache/src/internal/cacheUtils.ts#L86-L106)

```typescript
// Use zstandard if possible to maximize cache performance
export async function getCompressionMethod(): Promise<CompressionMethod> {
  if (process.platform === 'win32' && !(await isGnuTarInstalled())) {
    // Disable zstd due to bug https://github.com/actions/cache/issues/301
    return CompressionMethod.Gzip
  }

  const versionOutput = await getVersion('zstd')
  const version = semver.clean(versionOutput)

  if (!versionOutput.toLowerCase().includes('zstd command line interface')) {
    // zstd is not installed
    return CompressionMethod.Gzip
  } else if (!version || semver.lt(version, 'v1.3.2')) {
    // zstd is installed but using a version earlier than v1.3.2
    // v1.3.2 is required to use the `--long` options in zstd
    return CompressionMethod.ZstdWithoutLong
  } else {
    return CompressionMethod.Zstd
  }
}
```

- 圧縮オプションは`createTar()`で決めています。
  - [https://github.com/actions/toolkit/blob/main/packages/cache/src/internal/tar.ts#L86-L126](https://github.com/actions/toolkit/blob/main/packages/cache/src/internal/tar.ts#L86-L126)

```typescript
  function getCompressionProgram(): string[] {
    switch (compressionMethod) {
      case CompressionMethod.Zstd:
        return ['--use-compress-program', 'zstd -T0 --long=30']
      case CompressionMethod.ZstdWithoutLong:
        return ['--use-compress-program', 'zstd -T0']
      default:
        return ['-z']
    }
  }
  const args = [
    '--posix',
    ...getCompressionProgram(),
    '-cf',
    cacheFileName.replace(new RegExp(`\\${path.sep}`, 'g'), '/'),
    '-P',
    '-C',
    workingDirectory.replace(new RegExp(`\\${path.sep}`, 'g'), '/'),
    '--files-from',
    manifestFilename
  ]
```

というわけで、キャッシュ対象がキャッシュサイズ上限の5GBに収まるかを調べるには、
キャッシュ対象を`Gzip`もしくは`zstd`で圧縮した後のサイズを見れば良いことがわかります。

## キャッシュサイズ調査

次に、キャッシュ対象が上限の5GBに収まるかを調査します。
`Gzip`か`zstd`かは大した差ではないので、とりあえず`Gzip`で圧縮する場合を調査します。

キャッシュサイズ調査を進めるにあたって、何をキャッシュ対象とするかを考える必要があります。
ONNX-MLIRが依存するライブラリのうち一番大きいのは`llvm`ですので、これを対象とします。
キャッシュ対象として、(1)ソースコードとビルド済みバイナリの両方を含める、(2)ビルド済みバイナリのみ、2パターンが考えられるため、それぞれの圧縮後のサイズを調査します。

またONNX-MLIRはビルドシステムとして`CMake`を使用しており、ビルド時の`CMAKE_BUILD_TYPE`オプションを指定することで、
適切なオプションを付けたビルドが可能です。
デフォルトでは`Release`が指定されていますが、ビルド後のバイナリサイズを最小化する`MinSizeRel`を指定ことも可能です。

したがって、キャッシュ対象2パターン×`CMAKE_BUILD_TYPE`オプション2パターンの4パターンの圧縮後のサイズを調査しました。
以下の結果を見ると、どのパターンでも圧縮後のサイズは2GB以下であり、上限の5GB以下でした。

| キャッシュ対象 | CMAKE_BUILD_TYPE=Release | CMAKE_BUILD_TYPE=MinSizeRel |
| :--- | :---: | ---: |
| (1)ソースコードとビルド結済みバイナリの両方 | **1814 MB** | 1658 MB |
| (2)ビルド済みバイナリのみ| 629 MB | 681 MB |

そのため、デフォルトのビルドオプションに一番近い、かつ、キャッシュ対象が一番広い
「(1)ソースコードとビルド済みバイナリの両方かつ、`CMAKE_BUILD_TYPE=Release`」を採用することとします。

## CI設定の設計

CI設定は以下としました。

- Pythonバージョン
  - Python 3.7.8を使用
    - actions/pythonの制約で3.7.0が指定できないため
- ビルドスクリプト
  - 公式手順にしたがったビルド
- キャッシュ範囲の検討
  - キャッシュ対象
    - `llvm`のソースコードとビルド済みバイナリの両方
    - キャッシュキーは`clone-milr.sh`と`build-mlir.sh`の中身

この設定でクローンしたリポジトリでCIを実行したところ、問題なくビルドできることを確認しました。
またビルド時間を測定したところ、キャッシュにより7倍に高速化できました。

- キャッシュミス：1h 12m 0s
- キャッシュヒット：12m 37s

## 学び

- 公式ドキュメントに書いていないことがある
  - ソースコードを見ればわかる

## おわりに



# メモ


## キャッシュキー衝突によりCIが失敗する問題

- 問題の内容
  - 同時に複数のビルドが走ると、キャッシュキーの衝突が起きる
    - キャッシュキーの確保→キャッシュキーに対してコンテンツを格納
- 解決策
  - ビルドキャッシュをあきらめる
  - キャッシュキーにバージョン番号を含めて、衝突が起きたらバージョンを上げる
    - →メンテナーに提案したところ、後者で行くこととなった
