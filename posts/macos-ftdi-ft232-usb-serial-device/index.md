# FTDI社製FT232シリーズのUSBシリアル変換アダプタをmacOSで使う


12ステップで作る 組込みOS自作入門を再開するにあたり、MacBook Pro 13インチを購入したのでmacOS上で開発しようと思う。

使用するUSBシリアル変換アダプタは下記で購入したもの。

- [ＦＴ２３２　ＵＳＢシリアル変換ケーブル　ＶＥ４８８](http://akizukidenshi.com/catalog/g/gM-08343/)

# ドライバ導入

以下のページから`FTDIUSBSerialDriver_xxxxx.dmg`をダウロードし、ダブルクリックして開く。

- [Virtual COM Port Drivers](https://www.ftdichip.com/Drivers/VCP.htm)

中に入っている`FTDIUSBSerial.pkg`を右クリックし、Openをクリックする。インストーラが起動するので、指示に従いドライバをインストールする。

# 接続確認

シリアル変換デバイスをMacに接続し、iTermで`ls /dev/cu.*`を実行すると`/dev/cu.usbserial-XXXXXX`(`XXXXXX`は英数字)が表示される。

あとは`screen /dev/cu.usbserial-XXXXXX`でこのデバイスに繋ぐだけ。
