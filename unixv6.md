# OSコードリーディングを読む会 メモ

- http://simh.trailing-edge.com/

- C Reference Manual
  - https://www.bell-labs.com/usr/dmr/www/cman.pdf

## シミュレータ・ディスクイメージ
- 安定の@orumin氏のサイトが最も役に立つ。
  - http://orumin.blogspot.jp/2014/06/unix-and-2bsd-on-pdp11simh-1.html
  
- ソースはこれを落とすのがよさそう
  - ftp://ftp.ics.es.osaka-u.ac.jp/pub/mirrors/UnixArchive/PDP-11/Distributions/research/Dennis_v6/v6src.tar.gz

- カーネルは入っていないので結局ここから見ることになる
  - http://minnie.tuhs.org/cgi-bin/utree.pl?file=V6/usr/sys

- プログラミングサンプル
  - http://aiju.de/code/pdp11/programming
  
- https://blog.freedom-man.com/unix-v6-1/

- シミュレータはCTRL+EでBreakできる。そこでqを押せば終了できる。（syncを事前に3回くらい叩くこと！ディスクが壊れるかもよ！）


## 手動でテープインストール

- ここから入手できそう（distribution tapeをダウンロード）
  - http://doc.cat-v.org/unix/v6/operating-systems-lecture-notes/v6/

- あとは http://orumin.blogspot.jp/2014/06/unix-and-2bsd-on-pdp11simh-1.html を参考に進める

- `STTY -LCASE` を実行すれば、小文字が入力できるようになる。

- `/usr/s1/ed.c`
- `/usr/s2/sh.c`

- edの使い方
  - http://www.karing.jp/~yoshino/edtips.html
  - https://ja.wikipedia.org/wiki/Ed
  - https://www.gnu.org/software/ed/manual/ed_manual.html
  
`ed <filename>` で起動し、Enterを押し下げるごとに行が移動

`-2` とか `+5` とかすればその行数分移動する

`c`と入力すればその行を消して、そこに行を挿入するモードになる

入力が終了したら`.`のみを打ってEnterを押すと、コマンドモードに戻る。

`w`で書き込み、`q`で終了。

### telnetできない件
`/etc/ttys`を修正する。具体的には、tty0-8が無効化(先頭の数字が0にセット)されていたので、先頭の0を1にすればよい。




# 本の足りないところを埋めて行くメモ

## p.179 「システムコールは間接呼び出しもサポートしています」 -> どういうときに役立つ?
Lions本より
```
Indirect calls have a higher overhead than direct
system calls. Indirect calls are needed when
the parameters are data dependent and cannot be
determined at compile time.
```
ということで、「コンパイル時にパラメータが決定できない場合」に使うということらしい。


