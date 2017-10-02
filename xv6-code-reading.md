# OSコードリーディングを読む会 メモ

- http://simh.trailing-edge.com/

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


## 手動でテープインストール

- ここから入手できそう（distribution tapeをダウンロード）
  - http://doc.cat-v.org/unix/v6/operating-systems-lecture-notes/v6/

- あとは http://orumin.blogspot.jp/2014/06/unix-and-2bsd-on-pdp11simh-1.html を参考に進める

- `STTY -LCASE` を実行すれば、小文字が入力できるようになる。
