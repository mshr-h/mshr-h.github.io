# UNIXコマンド Tips


たまに使うUNIXコマンドとその使い方をメモ。

# zip

- `-0`
    - store only(圧縮しない)
- `-j`
    - junk (don't record) directory names(ディレクトリ名を保存しない)

# zipinfo

ZIPファイル内に格納されているファイルの一覧やサイズ、ファイルを表示する。

# basename

ディレクトリ名(not パス)を取得するコマンド。

例：現在のディレクトリ名を取得する。

```bash
basename "$PWD"
```

# qpdf

PDFファイルを操作するコマンド。

## macOSへの導入

```bash
$ brew install qpdf
```

## 実行例

`input.pdf`の暗号化を解除し、`output.pdf`として出力する。

```bash
$ qpdf --decrypt --password=1234 input.pdf output.pdf
```

## オプション

- `--decrypt`
    - 暗号化を解除
- `--password`
    - パスワードを指定
- -`-replace-input`
    - 入力ファイルを上書き

# stress-ng

```bash
stress-ng
```
