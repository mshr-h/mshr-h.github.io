# プロキシ環境下でNow CLIを使う


ZEITは、静的サイト/サーバレス向けPaaSの[Now](https://zeit.co/)を開発・提供している。NowのインタフェースとしてCLIツールが公式に開発されている。しかし、Now CLIはプロキシ環境下で使えないという制約がある。2017年1月27日に下記のGitHub Issueで話題に上がっているが、3年立った2020/3現在もProxyに対応していない。

- [Unable to deploy because of proxy · Issue #255 · zeit/now](https://github.com/zeit/now/issues/255)

そこで、Now CLIをプロキシ対応させるラッパーが非公式に開発されており、これを使うことでプロキシ環境下でもNow CLIを使うことができる。

- [proxify-now - npm](https://www.npmjs.com/package/proxify-now)

---

# 導入方法

以下のコマンドで導入できる。

```bash
$ npm i -g now global-agent proxify-now
```

# 使い方

環境変数の`http_proxy`と`https_proxy`にプロキシサーバを設定する。Now CLIを使う際に、コマンド名を`now`ではなく`pnow`を実行するだけ。
