# Windows 10のVirtualBoxで「Error In supR3HardNtChildPurify」とエラーが表示されて仮想マシンが起動しない問題


ある日、Windows上のVirtualBoxでVMを起動しようとしたら以下のエラーメッセージが出て起動しない問題が発生した。

![/img/post/2019-06-23-virtualbox.png](/img/post/2019-06-23-virtualbox.pngs)

Webで検索してみたところ、4年ぐらい前のバージョンでも同様の問題が起きていたそう。

- [#13697 (4.3.20 doesn’t start virtual machines any more (supHardenedWinVerifyProcess failed)) — Oracle VM VirtualBox](https://www.virtualbox.org/ticket/13697)

どうやらアンチウイルスソフトが問題の模様。今回、問題が起きたPCではウイルスバスターを導入しているが、これをアンインストールすることは諸事情によりできない。なので、VirtualBoxの古いバージョンを入れるしかなさそう。というわけで、VirtualBox 6.0.0をアンインストールし、VirtualBox 5.2.2を使用することで起きなくなった。

