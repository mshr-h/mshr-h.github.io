# ONNX-MLIRにmacOSのCI設定を追加したときの試行錯誤メモ


2020年10月、[ONNX-MLIR](https://github.com/onnx/onnx-mlir)という機械学習コンパイラ（ONNXモデルを実行可能バイナリへ変換するツール）に
[macOSのCI設定を追加するPR](https://github.com/onnx/onnx-mlir/pull/287)を送り、マージされました。
本記事では、このPRを作成してからマージされるまでの試行錯誤を紹介します。

## 経緯

OSS機械学習コンパイラの調査の一環でONNX-MLIRを見つけ、開発用PC(macOS; MacBook Pro 13")に導入していました。
リポジトリ内を見ていたところ、x86-Linux、s390-Linux、x86-WindowsのCIは走っていますが、どうやらmacOSのCIはないみたいでした。
Issueに「macOS向けCIパイプラインを追加しようと思ってるけどどう？」と[聞いてみた](https://github.com/onnx/onnx-mlir/issues/252)ところ、
メンテナの一人から是非やってやって欲しい旨のコメントがあり、作業に着手しました。

## CIサービスの選定

GitHubをサポートしているCIサービスはいくつかありますが、メンテナからはTravis CIでやって欲しいとの要望があったため、
Travis CI向けの調査に着手しました。

ONNX-MLIRでは、ビルドにDockerを使っています。
単一のDockerイメージでビルドするのではなく、以下のようにビルドを2段階に分ける構成とすることで、
依存ライブラリのビルド成果物を再利用可能となり、ビルド時間の短縮を実現しています。

1. 依存ライブラリをビルドするDockerイメージ
   - `llvm/mlir`が一番大きな依存ライブラリで、ビルドに1時間程度かかる
   - 依存ライブラリの更新時にビルドされる
2. 上記でビルドした生成物を使ってONNX-MLIR本体をビルドするDockerイメージ
   - このイメージはTravis-CI実行時に毎回ビルドされる

Travis CIとDockerのmacOSサポート状況を調査したところ、

- DockerはゲストOSとしてのmacOSをサポートしていない（多分）
- Travis CIのmacOS環境はDockerをサポートしていない（[Using Docker in Builds](https://docs.travis-ci.com/user/docker/)）

ため、Travis CIは使えないと判断しました。

ただ、実現したいことは「キャッシュ機構による高速化」であることは分かった、代替案の調査に着手しました。

別のCIサービスとしてGitHub Actionsを調査したところ、[キャッシュ機構](https://docs.github.com/en/actions/guides/caching-dependencies-to-speed-up-workflows)(`actions/cache@v2`)をサポートしており、これを提案しようと考えました。
（今考えると、Travis CIのキャッシュ機構を使えば同じことができますね。。。）

しかし、GitHub Actionsのキャッシュ機構は以下の制約があるとわかりました。

1. 7日間以上アクセスのないキャッシュエントリは削除される
2. リポジトリ全体でキャッシュサイズの上限が5GB

上記の制約があるが、GitHub Actionsは使えそうとのことをメンテナに相談したところ、
`1.`の制約は問題なく、`2.`が大丈夫ならGitHub Actionsを使おうとのコメントをもらいました。
というわけで、キャッシュ対象のファイルサイズの調査をします。

## `actions/cache@v2`調査

キャッシュ対象のファイルサイズの調査の前に、どのようにキャッシュされているか知るために`actions/cache@v2`を調査します。
GitHub Actionsは、リポジトリ全体でキャッシュサイズの上限が5GBという制約があります。
実はこのサイズは、キャッシュ対象の圧縮後のサイズを指しています。（公式ドキュメントには記載がなく、ソースコードを読むとわかりました）

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
macOSの場合は`zstd`を使うようなので、`zstd`による圧縮した後のサイズを見ます。

キャッシュサイズ調査を進めるにあたり、キャッシュ対象を考える必要があります。
ONNX-MLIRが依存するライブラリのうち一番大きいのは`llvm`のため、これを対象とします。
キャッシュの範囲としては、(1)ソースコードとビルド済みバイナリの両方を含める、(2)ビルド済みバイナリのみ、2パターンが考えられるため、それぞれの圧縮後のサイズを調査します。

またONNX-MLIRはビルドシステムとして`CMake`を使用しており、ビルド時の`CMAKE_BUILD_TYPE`オプションを指定することで、
適切なオプションを付けたビルドが可能です。
デフォルトでは`Release`が指定されていますが、ビルド後のバイナリサイズを最小化する`MinSizeRel`を指定ことも可能です。

以上より、キャッシュ範囲2パターン×`CMAKE_BUILD_TYPE`オプション2パターン=4パターンの圧縮後のサイズを調査しました。
以下の結果を見ると、どのパターンでも圧縮後のサイズは2GB以下であり、上限の5GB以下でした。

| キャッシュ対象 | CMAKE_BUILD_TYPE=Release | CMAKE_BUILD_TYPE=MinSizeRel |
| :--- | :---: | ---: |
| (1)ソースコードとビルド結済みバイナリの両方 | **1814 MB** | 1658 MB |
| (2)ビルド済みバイナリのみ| 629 MB | 681 MB |

そのため、デフォルトのビルドオプションに一番近い、かつ、キャッシュ対象が一番広い
「(1)ソースコードとビルド済みバイナリの両方かつ、`CMAKE_BUILD_TYPE=Release`」を採用することとします。

## ワークフローの設計

GitHub Actionsでは、実行する処理とその処理の実行条件を定義したものをワークフローと呼び、YAML形式で記述します。
今回のワークフローは以下としました。

- Pythonバージョン
  - Python 3.7.8を使用
    - actions/pythonの制約で3.7.0が指定できないため
- ビルド手順
  - 公式が提供するビルドスクリプト群を使用
- キャッシュ対象
  - `llvm`のソースコードとビルド済みバイナリの両方
  - キャッシュキーは`clone-milr.sh`と`build-mlir.sh`の中身

クローンしたリポジトリでこのワークフローをテスト実行したところ、問題なく完了しました。
複数回実行してキャッシュの効果を確認したところ、キャッシュにより7倍の高速化を確認しました。

- キャッシュミス：1h 12m 0s
- キャッシュヒット：12m 37s

## PR提出

細かな修正をワークフローに加えて、PRを提出しました。

メンテナ側でテストしたところ、キャッシュの保管が失敗した際に、キャッシュキーが予約状態のままワークフローが終了してしまう問題が起きました。
対策として、`actions/cache`の[Issueで紹介されていた](https://github.com/actions/cache/issues/144)キャッシュキーを手動で書き換えるワークアラウンドを適用し、問題が解決されました。

メンテナ側のテストでOKとなり、PRがマージされました。

## おわりに

本記事では、ONNX-MLIRにmacOSのCI設定を追加した際の試行錯誤を紹介しました。
OSSメンテナとコミュニケーションを取りながらPRを作成し、無事にマージされました。
この体験は非常に楽しく、自身のモチベーションが上がるものでした。
今後も継続してOSSへ貢献していきたいです。

