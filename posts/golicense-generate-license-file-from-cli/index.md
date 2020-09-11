# LICENSEファイルを生成するgolicenseを作った


[mshr-h/golicense](https://github.com/mshr-h/golicense)

golicenseという、コマンドライン上でLICENSEファイルを生成するプログラムをGo言語で作成した。いくつかのライセンスから選んで、そのライセンスファイルをカレントディレクトリに出力する。

現在サポートしているライセンスは以下の3つのみ。

- MIT
- GPL-3
- Apache 2.0

# インストール方法

```
go get github.com/mshr-h/golicense
```

# おわりに

Goの標準ライブラリのみ使用可能、という縛りで実装してみた。この程度の機能なら、外部ライブラリに頼らずとも簡単に実装できることが分った。他にもコマンドラインツールを作成して、Go標準のライブラリの勉強を進めていきたい。
