# プロキシ環境下でWindows 10のライセンス認証する


# TD;LR

```
> netsh winhttp set proxy 1.2.3.4:5678
> slmgr.vbs -ipk AAAAA-BBBBB-CCCCC-DDDDD-EEEEE
```

# 手順

PowerShellを管理者権限で起動する。
`netsh winhttp show proxy`で現在のプロキシ設定を確認する。

```
PS C:\\Windows\\system32> netsh winhttp show proxy

現在の WinHTTP プロキシ設定:

    直接アクセス (プロキシ サーバーなし)。
```

`netsh winhttp set proxy 1.2.3.4:5678`でプロキシを設定する。下記コマンド実行例は、プロキシサーバが`1.2.3.4:5678`である前提。

```
PS C:\\Windows\\system32> netsh winhttp set proxy 1.2.3.4:5678

現在の WinHTTP プロキシ設定:

    プロキシ サーバー:  1.2.3.4:5678
    バイパス一覧     : (なし)
```

再度プロキシ設定を確認し、正しく設定されていることを確認する。

```
PS C:\\Windows\\system32> netsh winhttp show proxy

現在の WinHTTP プロキシ設定:

    プロキシ サーバー:  1.2.3.4:5678
    バイパス一覧     : (なし)
```

`slmgr.vbs -ipk AAAAA-BBBBB-CCCCC-DDDDD-EEEEE`でライセンス認証をする。下記コマンド、実行例はプロダクトキーが`AAAAA-BBBBB-CCCCC-DDDDD-EEEEE`である前提。

```
PS C:\\Windows\\system32> slmgr.vbs -ipk AAAAA-BBBBB-CCCCC-DDDDD-EEEEE
```

![/img/post/2020-02-08-windows-activation.png](/img/post/2020-02-08-windows-activation.png)

以上の手順でライセンス認証は完了。
