# NVMe関連情報
## nvme-cli
- いろいろ見れる便利コマンド
- https://github.com/linux-nvme/nvme-cli
- たとえばSMART情報: `sudo /usr/local/sbin/nvme smart-log /dev/nvme0`
pec: http://www.nvmexpress.org/wp-content/uploads/NVM_Express_Revision_1.3.pdf

SQ: Submission Queue
PRP: Physical Region Page
SGL: Scatter Gather List
NS: NameSpace
CNS: Controller or Namespace Structure


Submission Queue と Completion Queue がセットで用いられる。
これらはメモリ上に確保される。

Admin (Submission | Completion) Queue コントローラの管理に用いられる。
I/O (Submission | Completion) Queue をつくったり、コマンドを中止したりできる。

たいてい、各コアごとにひとつI/O Queue Pairを作成する（キャッシュとかが正しく働くように&ロックで悩まないで済むように）
cf. p.8 Queue Pair Example

SQ の各エントリはコマンドで、ひとつあたり64bytes。リングバッファ。
4.2にその構造がある。
このうち、OPC（OPCode ）は

NS(NameSpace)はNVMe上の領域を切り分けてそれに名前をつけられる機能。

p.61 Figure 25: SGL Read Example

p.74	5 Admin Command Set
			(Create | Delete) IO (Submisson | Completion) Queue
p.172	6 NVM Command Set
			Read / Write / Flush

7.2.1 Command Processing
	

Figure 251: Command Processing

7.2.2 Basic Steps when Building a Command

7.2.5.1 Creating an I/O Submission Queue

Figure 252: PRP List Describing I/O Submission Queue
	PRP ListでどのようにQueueを定義するかの図解

7.6 Controller Initialization and Shutdown Processing
7.6.1 Initialization


概要まとめ
- コマンドのやり取りはSQ/CQのペアで行う
- SQ/CQは、メモリ上に配置される。


## アクセス手順まとめ

### 共通：コマンドの送り方
7.2.1 Command Processing を参照
Figure 251: Command Processing にイメージ図あり

- 送りたいコマンドをSQの末尾に追加する
  - コマンドのフォーマットは以下を参照
    - 4.2 Submission Queue Entry – Command Format
  - メモリ領域を指定する場合はPRPEntry(List)もしくはSGLを使用する
- Submission Queue Tail Doorbell Register (Controller Registersの中にある)を更新する。すると、コントローラがコマンドを処理してくれる。
- コマンドの実行順序はOutOfOrder
- 実行が終わるとコントローラがCQの末尾にエントリを追加する。エントリのPhase Tagが反転するので、ホストは新しいエントリかどうか判別できる。
- （オプション）割り込みを発生させる。設定によっては、毎回発生するわけではない。
- ホストはCQの各エントリの処理が終わったら、Completion Queue Head Doorbell registerを設定し直す。

QueueのHeadとTailがどこを指すべきかは

4.1.1 Empty Queue	Figure 8: Empty Queue Definition
4.1.2 Full Queue	Figure 9: Full Queue Definition
に図解がある。

## uioのメモ
- `/sys/class/uio/uio0/device/resource0 or 1`をmmapすればBAR0, 1にアクセスできる
- 割り込みはpin-based. 
  - https://www.kernel.org/doc/html/v4.12/driver-api/uio-howto.html#how-uio-works
  - 設定はuioの方でやってくれるので、あとは使うだけ

## 疑問点
- completionQueueのサイズは16bytesでOK?（4.6 Completion Queue Entryには、at least 16bytesと言っていて、将来は変わるかもよと言っている。たぶん今は16なのだろう。）

### 初期化
7.6.1 Initialization を参照
- デバイスをみつける
- <BAR0, 1>を読んで、Controller Registersがマップされている場所を知る
  - Controller Registersの内容は3.1 Register Definitionにある
- 割り込みの設定をする
  - pin-based (uioではこちら)
	- MSICAP.MC.MSIE=’0’
    - 7.5.1.1.1 Interrupt Example (Informative)  を参照
      - CQyHDBLを更新することでコントローラに割り込み処理終了を通知できる
  - single-MSIをつかう場合
	- MSICAP.MC.MSIE=’1’
	- MSICAP.MC.MME=0h
    - 7.5.1.1.1 Interrupt Example (Informative)  を参照
      - CQyHDBLを更新することでコントローラに割り込み処理終了を通知できる
- CSTS.RDYが0になるまで待つ。
- まずはAdmin Queueを設定する。
  - ASQ, ACQのIdentifierは0
  - 3.1.8 Offset 24h: AQA – Admin Queue Attributes
    - ASQ, ACQの大きさ（エントリ数）を設定
    - とりあえず8にでもしてみよう
	- ASQの1エントリは64bytes. 4.2のFigure2を参照。
	- SQの1エントリは16bytesっぽい（将来は変わるかもと書いてある。CC.IOCQESを読んだらそうなっていたことも確認した。）
  - 3.1.9 Offset 28h: ASQ – Admin Submission Queue Base Address
    - ASQの物理アドレスを設定
  - 3.1.10 Offset 30h: ACQ – Admin Completion Queue Base Address
    - ACQの物理アドレスを設定
    - ACQの領域は0クリアしておく(Phase Tagを0にしたいから)
- 各レジスタを設定する
	- CC.AMS
      - 000b: Round Robin(とりあえずこっち)
      - 001b: Weighted Round Robin with Urgent Priority Class
    - CC.MPS
      - ページサイズ(bytes) == (2 ^ (12 + MPS)) となるように設定
      - 0でよさそう(4KBページなら)
    - CC.CSS
      - 000b: NVM Command Set に設定
- コントローラを有効化(CC.EN = 1)
- CSTS.RDY == 1 になるまで待つ
- Identifyコマンドを発行して、コントローラとNSの情報を取得する
  - 5.15 Identify command
  -　コマンドの発行自体は7.2を参照
- 7.4.1 Queue Setup and Initialization に従ってQueueを設定する
  - 5.21.1.7 Number of Queues (Feature Identifier 07h) コマンドを発行して
	各Queueを何個つくるかコントローラに設定。
  - 各キューを各コマンドで作成
    - 5.3 Create I/O Completion Queue command
    - 5.4 Create I/O Submission Queue command

### シャットダウン手順
- 7.6.2 Shutdown を参照

### Submission / Completion queue
- CAP.DSTRDってなんぞや: 読んでおいて使えばOK。たぶんハードウエア的な制約に合わせるための値

### コマンドの送り方
- 7.2を参照
- 詳しい手順は 7.2.2 Basic Steps when Building a Command
- 7.2.2 Figure 251に図がある
- SQの次の空きスロットにコマンドを書き込む
  - 空きスロット = SQyTDBLの値

### Identify command
CDW0.OPC
CDW0.FUSE
CDW0.CID = テキトーに（今回はシリアルな番号を振った）
CDW1.NSID = ?
MPTR : not used
PRP1 = PRPEntry or PRPList へのポインタが来る。どっちだ？
8pにEach command may include two PRP entries or one Scatter Gather List (SGL) segment. If more than two PRP entries are necessary to describe the data buffer, then a pointer to a PRP List that describes a list of PRP entries is provided. と書いてある。
だから、PRP Entryということかね？
PRP2 

Figure 108: Identify – Command Dword 10
これもつかうって！


### PRP Entry / List


### 死ぬ！
CSTS.CFS=1になってリセットが効かなくなってしまった。Completion Queueが正しくないということのようだ。
10.5 Controller Fatal Status Condition
いやどうやってリセットするのさ!!!

原因: バスマスタを有効にしていなかった。


### 割り込み来ない！
実機ではうまく行った。以下はそのログ。
```
$ make run
sudo sh -c "echo 120 > /proc/sys/vm/nr_hugepages"
sudo ./a.out
vid:00008086 did:0000F1A5
PCI.CAP=40
PCI.STS=0010
PCI.CMD=0006
MSI.=0005
CAP: TO=120 NSSRS=1  
CC: EN=1 CSS=0 MPS=0 AMS=0 SHN=1 IOSQES=6 IOCQES=4  
CSTS: RDY=1 CFS=0 SHST=2 NSSRO=0 PP=0  
Performing reset...
ASQ @ 0x7f294fc00000, physical = 0x412200000
ACQ @ 0x7f294fa00000, physical = 0x412000000
ASQ @ physical 0x412200000 (size = 8)
ACQ @ physical 0x412000000 (size = 8)
CC: EN=0 CSS=0 MPS=0 AMS=0 SHN=0 IOSQES=0 IOCQES=0  
CSTS: RDY=0 CFS=0 SHST=0 NSSRO=0 PP=0  
Enabling controller...
Controller initialized.
CC: EN=1 CSS=0 MPS=0 AMS=1 SHN=0 IOSQES=0 IOCQES=0  
CSTS: RDY=1 CFS=0 SHST=0 NSSRO=0 PP=0  
main thread!!
Interrupted!

```

http://www.hep.by/gnu/kernel/uio-howto/uio_pci_generic_internals.html

On each interrupt, uio_pci_generic sets the Interrupt Disable bit. This prevents the device from generating further interrupts until the bit is cleared. The userspace driver should clear this bit before blocking and waiting for more interrupts.

http://www.hep.by/gnu/kernel/uio-howto/uio_pci_generic_example.html

https://github.com/rumpkernel/wiki/wiki/Howto:-Accessing-PCI-devices-from-userspace


https://os.inf.tu-dresden.de/ddekit/dde_rtlws11.pdf

https://github.com/spotify/linux/blob/master/drivers/uio/uio_pci_generic.c


以下のコマンドで割り込みをリアルタイムに確認できる。
```
watch -n1 "cat /proc/interrupts"
```

これを確認したところ、割り込み自体はOSまで届いているようだった。
ただし、ハンドルされずに増えて止まった（OS側でdisableにされた旨のメッセージがdmesgに記録される。）



### リビルドすればなおる？ from Liva
https://github.com/PFLab-OS/Raph_Kernel_devenv_box/blob/master/uio.sh


### 割り込みこない 状況を整理

- mars
	- 

- qemu(kvm) on mars
	- `ssh -L 5900:localhost:5900 mars`
	- xhci_uioは動作
	- nvme_uioは動作しない
		- 割り込みが1回も発生しない。
		- 実行後はコントローラーが死んでしまう(EN=0にはなるが、EN=1にできない) 

- qemu on t03
	- xhci_uioは動作
	- nvme_uioは動作しない
		- 割り込みが1回も発生しない。
		- 実行後はコントローラーが死んでしまう(EN=0にはなるが、EN=1にできない, CFS=1になっている) 

- virtualbox on t03
	- nvme_uioは動作しない
		- 割り込みは発生しており、 `/proc/interrupts` でも確認できるが、それをuioで正しく取れていない。
		- dmesgでも `irq 22: nobody cared` と表示され、割り込みが無効化されてしまう。
			- `make restore && make load` すれば割り込みは再度有効になり、プログラムも実行できるが状況は変わらず。

### 可能性をつぶす
- queueが正しく設定されていない
	- それはない。以前はそれでバグっていたが修正された。
	- そもそも、コマンドの送信を行ってからIRQが飛んできていることは確認できている。
	- したがって、Submission Queueは少なくとも正しい。
- Completion Queueはどうだろうか？

		












