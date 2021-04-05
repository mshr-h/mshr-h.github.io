# Weekly Report(2021/04/04)


今週の振り返り。

## 英語

- 今週は7回レッスンを受けた
  - 満足
- 感情を表す語彙力が少なく、いい相づちを打てなくてモヤモヤする
  - "That's nice"、"Sounds good"などは出てくるが、バリエーションが少ない
- Whyに対して答えるとき、返答に時間がかかる時がある
  - 結局何が言いたいかまとまらない
- 英会話力を上達するには、「言いたいことを事前に文章に落とす→英会話の場で使ってみる」を繰り返すのが近道なのでは、ということに気づいた

## Machine learning / Cloud computing

- TVMのビルドで、LLVMをstatic linkしたほうが良さそう
  - cmakeのconfig時に、`USE_LLVM("llvm-config --link-static")`を指定すればstatic linkされる

## Rock climbing

- MoonBoardのBenchmark V5~V7を触るなど
- 登っているとき小指が使えてないことに気付いた
  - なるべく中指、薬指、小指の3本で登るようにする
- 外岩行きたい

## その他トピック

- [HPy - A better C API for Python](https://lwn.net/SubscriberLink/851202/8981fa354a584aeb/)
  - Python C APIの次世代版のようなもの
  - debug modeが充実、universal binary(HPyでC拡張を実装すれば、同じバイナリをCPython、PyPy、GraalPythonなどロードできる)など

