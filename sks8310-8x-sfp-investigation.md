# SKS8310-8X：U-Boot での SFP+ 認識・リンク調査

調査日：2026-11-01（日本時間）

## 結論

ポート1に挿入された **FiberJP `SFP-10GBX23-10`** の識別 EEPROM と診断情報を、U-Boot の I²C コマンドから取得できた。EEPROM の基本・拡張チェックサムは両方一致した。

調査中にスイッチ側で **10Gbps・全二重のリンクアップ** を確認し、時間を置いた再確認でも維持されていた。ユーザーからも対向ルーターが「up になった」と報告があり、両端のリンク成立を確認できた。

`rtk network on` によるスイッチ機能の初期化後にリンクアップしたことから、U-Boot 側の初期化不足が原因だった可能性が高い。初期化・I²C 設定・経過時間の影響を切り分けずに確認した結果のため、原因については推定である。

## 調査対象

| 項目 | 内容 |
|---|---|
| スイッチ | SKS8310-8X |
| U-Boot のボードモデル | `RTL9303_8XGE_V1` |
| 初期化時の検出チップ | RTL9303 |
| U-Boot | `2011.12.(3.6.11.55242)` |
| U-Boot ビルド日時 | `Mar 20 2026 - 11:31:17` |
| シリアル接続 | `/dev/ttyUSB0`、115200 baud、8N1 |
| 通常の接続方法 | `minicom -w -D /dev/ttyUSB0` |
| 調査ポート | 前面ポート1（MAC ポート0、SerDes 2 の対応は OpenWrt の機種定義を参照） |
| モジュール | 10G BiDi SFP+ |
| 対向 | ルーター。ユーザー申告では10G設定済み |

当初の症状は「トランシーバーを挿しても対向ルーター側でリンクアップしない」だった。

## 調査の経過

1. U-Boot の `help rtk` と `version` を確認した。
2. 初回調査では `rtk phyreg get 0 0 1` が `0xf00e`、`rtk i2c read 0 0` が `61455` を返した。
3. `rtk sdsreg get 2 0x1f 9` の実行後に U-Boot の応答が止まり、Ctrl+C でも復帰しなかった。ユーザーが電源を入れ直して U-Boot に復帰させた。
4. 復帰後、I²C の初期設定は `61456` で失敗した。
5. `rtk network on` を実行し、RTCORE、GPIO、I²C、RTL8231、MAC、PHY、SerDes などの初期化ログと `Chip 9303 (found)` を確認した。
6. この段階でも `rtk i2c read 0 0` は `61455` だったため、ポート1用の I²C デバイス設定を作成した。その後、識別 EEPROM の読み取りに成功した。
7. 診断領域も読み、温度・電圧・送受信光量・障害フラグを取得した。
8. MAC リンク状態は最初 `0x00000000` だったが、その後 `0x10000001` に変化した。速度・全二重状態も確認し、対向側の up 報告と一致した。

復帰後は各コマンドの応答とプロンプトの復帰を確認してから次へ進めた。既存の minicom がある場合は一時停止してシリアル応答の競合を避け、確認後に再開した。調査終了時も minicom の再開を確認済み。

## モジュールの識別結果

I²C アドレス `0x50` の先頭96バイトを読み取った。

| 項目 | 結果 |
|---|---|
| 識別子（byte 0） | `0x03`（SFP） |
| メーカー（bytes 20–35） | `FiberJP` |
| 型番（bytes 40–55） | `SFP-10GBX23-10` |
| CC_BASE（byte 63） | 格納値 `0xb0`、bytes 0–62 の加算下位8ビットも `0xb0` |
| CC_EXT（byte 95） | 格納値 `0x4e`、bytes 64–94 の加算下位8ビットも `0x4e` |

メーカー資料によると、この型番は送信1270nm／受信1330nm、10km用の10G BiDiモジュールである。波長には資料の仕様値を記載した。[メーカー仕様書](https://www.fiberjp.com/Content/files/Specifications/79/SFP-10GBX23-10%201270tx1330rx%2010km.pdf)

## 診断情報（DDM）

I²C アドレス `0x51` から取得した。識別 EEPROM の診断種別 byte 92 は `0x68` で、内部校正の診断値として換算した。以下は取得時点の値である。

| 項目 | オフセット（十進） | 生データ | 換算結果 |
|---|---:|---|---:|
| 温度 | 96–97 | `0x2fd9` | 約47.85℃ |
| 電源電圧 | 98–99 | `0x8281` | 3.3409V |
| TX バイアス電流 | 100–101 | `0x4ab5` | 38.25mA |
| TX 光出力 | 102–103 | `0x1a29` | 約−1.74dBm |
| RX 光入力 | 104–105 | `0x20ce` | 約−0.76dBm |
| 状態 | 110 | `0x08` | TX_DISABLE、ソフトウェア TX_DISABLE、TX_FAULT、RX_LOS はすべて0 |
| アラーム | 112–113 | `0x0000` | フラグなし |
| 警告 | 116–117 | `0x0000` | フラグなし |

換算式：温度は符号付き16ビット値 ÷ 256、電圧は値 × 100µV、バイアスは値 × 2µA、光量は値 × 0.1µW。光量をmWに直して `10 × log10(mW)` でdBmへ換算した。

これらはモジュールが報告する値である。受光とデータリンク成立を区別するため、別途 MAC の状態を確認した。

## リンク状態の実測

`md.l` で読み取った RTL930x のレジスターを、OpenWrt のドライバー定義に基づいて解釈した。

| コマンド | 最終実測値 | ポート1についての解釈 |
|---|---|---|
| `md.l 0xbb00cb10 1` | `10000001` | MAC ポート0の bit 0 が1：リンクアップ |
| `md.l 0xbb00cb18 1` | `00000004` | MAC ポート0の速度フィールドが4：10Gbps |
| `md.l 0xbb00cb28 1` | `1f110101` | MAC ポート0の bit 0 が1：全二重 |

リンク・速度レジスターは時間を置いて読み直しても同じだった。対向ルーター側のリンクアップはユーザーが確認した。今回確認した範囲は両端のリンク成立までである。

なお、`md.l 0xbb000194 1` はリンク前後とも `007df7df` だった。SerDes 2 に対応するフィールドだけを見ると `0x1f` だが、実際にはリンクアップしていた。この値から SerDes の停止を判定するには、強制設定なども含めて実効モードを調べる必要がある。

## 今回成功した確認手順

以下は、この機体・U-Boot・前面ポート1で実測した手順である。各コマンドの完了を確認してから次を実行する。

### 1. U-Boot のネットワーク機能を初期化

```text
rtk network on
```

これは状態を変更する操作で、スイッチ機能や PHY などの初期化・有効化を行う。U-Boot で再起動直後からリンクを使う場合は、まずこの初期化を確認する。

### 2. ポート1の識別 EEPROM を読む

```text
rtk i2c init hw 0 0 0 8 80 0 0
rtk i2c read 0 0
```

今回の成功時の応答：

```text
data[0] 0x3 / data[1] 0x4
```

`80` は I²C アドレス `0x50` の十進表記。デバイスID 0、インターフェース0、SDA選択0を使用した。OpenWrt の `i2c_mst1` という名前に対し、この U-Boot では上記のインターフェース番号で読み取りに成功した。

この U-Boot は1回の `read` で指定位置から2バイトを返した。先頭96バイトはオフセット `0, 2, 4, …, 94` で取得した。メーカー名は20–35、型番は40–55にある。

### 3. 同じモジュールの診断領域を読む

```text
rtk i2c init hw 1 0 0 8 81 0 0
rtk i2c read 1 96
rtk i2c read 1 98
rtk i2c read 1 100
rtk i2c read 1 102
rtk i2c read 1 104
rtk i2c read 1 110
rtk i2c read 1 112
rtk i2c read 1 116
```

`81` は `0x51` の十進表記である。ここでのデバイスID 1は、**ポート1の診断領域を読むために割り当てたID**である。別の既存設定がある環境ではIDの衝突を確認する。

初期設定時の `r9300_i2c_init had already been initialized!` という表示の後でも、今回の読み取りは成功した。

### 4. MAC のリンク・速度・全二重状態を確認

```text
md.l 0xbb00cb10 1
md.l 0xbb00cb18 1
md.l 0xbb00cb28 1
```

## エラーと、ヘルプ・公開実装の不一致

### PHY 読み取りエラーはモジュール不在を意味しなかった

```text
rtk phyreg get 0 0 1
Get Unit 0 Port 00 page 00 reg 01    Failed!! 0xf00e
```

このエラーは初期化後にも発生したが、その後10Gリンクアップを確認している。この機体で SFP 認識・リンク状態を判定するには、当該 PHY コマンドに加えて別の確認方法が必要である。識別情報は I²C、リンク状態は MAC レジスターで確認した。

公開 SDK のエラー定義との対応：

| 値 | 定義 | 解釈 |
|---|---|---|
| `0xf00e` / 61454 | `RT_ERR_SMI` | SMI エラー |
| `0xf00f` / 61455 | `RT_ERR_NOT_INIT` | ソフトウェアモジュールが未初期化である。SFPの有無は別途確認する |
| `0xf010` / 61456 | `RT_ERR_CHIP_NOT_FOUND` | チップ未検出。今回はスイッチ機能の初期化前に I²C 初期設定で発生 |

### `sdsreg get` のヘルプと引数処理

実機ヘルプには次の形式が記載されていた。

```text
rtk sdsreg get [sdsId] [page] [reg]
```

一方、参照した公開 SDK の `uboot_cmd.c` は `argIdx = 3` の後、`argv[++argIdx]` から3つの値を読む実装だった。ヘルプに従うと値は `argv[3]` から並ぶため、1つずれて末尾の引数を越えて参照する不具合がある。

この不具合は、実機で `rtk sdsreg get 2 0x1f 9` を送った際に応答が止まった原因の候補である。この推測は公開コードに基づくものであり、原因を特定するには実機バイナリーの引数処理を調べる必要がある。ダミー引数を足す回避策は、今回の調査では試さずに終えた。ユーザーがスイッチの電源を入れ直した後は、このコマンドの使用を避けた。

### `ext-pinGet` による GPIO 方向設定の可能性

ヘルプの説明は「GPIO 状態の取得」だが、公開 SDK の `rtk_extGpio_pin_get()` は入力方向を設定する `drv_generalCtrlGPIO_pin_init()` を呼んでから値を読む。

初回調査では、ポート1に対応する pin 0・1・2 の読み取り値はすべて0だった。信号の定義上は、挿入あり・送信停止なし・受信信号喪失なしに相当する。公開 SDK の実装から、`ext-pinGet` が GPIO の方向設定を変更した可能性がある。そのため、これらはコマンド実行後の値として扱う。

今回はユーザーによる電源再投入後に調査を再開し、`ext-pinGet` を再使用せず、EEPROM・DDM・MAC 状態から最終結論を得た。

### 数値の基数と I²C 応答形式

参照した SDK の I²C 引数処理は十進数指定である。ヘルプでは基数の説明が省略されている。成功手順はアドレスを `80` / `81` として記録した。初期化前の `0x50` 表記の試行では、別の初期化エラーも発生した。

また、参照した SDK の `rtk_i2c_read()` は1バイトを表示する実装だったが、実機は `data[0]` と `data[1]` の2バイトを返した。解析は実機の出力形式に合わせた。

## 操作範囲と終了状態

- `rtk network on` と、I²C デバイスの一時的な読み取り設定を実施した。
- 初回の `ext-pinGet` は上記の副作用の可能性がある。応答停止後の電源再投入はユーザーが実施した。
- モジュール EEPROM へのデータ書き込み、`saveenv`、フラッシュ書き込みはいずれも今回の操作から除外した。
- `rtk 10g ... fiber10g` による速度指定や、`rtk sdsreg set` などによる直接のレジスター書き換えも、今回の操作から除外した。ただし、`rtk network on` に含まれる内部の初期化処理は実行されている。
- 終了時は U-Boot が応答し、既存 minicom を再開済み。スイッチ・対向ルーターの双方でリンクアップを確認済み。

## 添付資料（本文内収録）

この文書だけで調査結果と作成資料を参照できるよう、実測ログ、EEPROM 抽出データ、調査用スクリプトを以下に収録した。資料はすべてこの文書内で読める。実測ログの内容は個体シリアル番号のマスクを除いて保持し、Markdown 掲載のため改行を LF に統一した。

SFP EEPROM bytes 68–83（個体シリアル番号）は、ログでは `[REDACTED_SERIAL]`、16進ダンプでは `XX` でマスクした。機器固有の MAC アドレスの記載はなかった。MAC ポート番号・MAC 状態レジスターは、ポートの対応や動作状態を確認する情報として保持した。型番・日付・測定値も保持した。チェックサムの一致はマスク前の原データで確認した。

### A. 電源再投入前の実測出力（会話記録からの抜粋）

初回調査はログファイルへの保存を開始する前だった。以下には会話に残った実測出力を抜粋して掲載した。記録の範囲はこの抜粋に限られる。GPIO 値の解釈には、前述した `ext-pinGet` の副作用に関する訂正が適用される。

```text
>>> 'rtk boardmodel'
rtk boardmodel
Board Model: RTL9303_8XGE_V1
RTL9300# #

>>> 'rtk ext-pinGet 0'
rtk ext-pinGet 0
pin0:	0

RTL9300# #

>>> 'rtk ext-pinGet 1'
rtk ext-pinGet 1
pin1:	0

RTL9300# #

>>> 'rtk ext-pinGet 2'
rtk ext-pinGet 2
pin2:	0

RTL9300# #

>>> 'rtk phyreg get 0 0 1'
rtk phyreg get 0 0 1
Get Unit 0 Port 00 page 00 reg 01    Failed!! 0xf00e
RTL9300# #

>>> 'rtk i2c read 0 0'
rtk i2c read 0 0
FAIL (61455)
RTL9300# #
```

この後、`rtk sdsreg get 2 0x1f 9` はコマンドのエコーまで返したがプロンプトが戻らず、Ctrl+C にも応答しなかった。その後の復帰操作はユーザーによる電源再投入だった。

### B. 復帰後のシリアル実測ログ（保存済みログの全内容）

ログは時刻情報を含まず、正確なリンク確立時間は不明である。以下に保存済みログの全内容を収録した。

```text

>>> version
version

U-Boot 2011.12.(3.6.11.55242) (Mar 20 2026 - 11:31:17)
mips-linux-uclibc-xgcc (Realtek MSDK-4.8.5p1 Build 2536) 4.8.5 20150209 (prerelease)
GNU ld (Realtek MSDK-4.8.5p1 Build 2536) 2.24.0
RTL9300# # 
>>> rtk i2c init hw 0 1 0 8 0x50 0 0
rtk i2c init hw 0 1 0 8 0x50 0 0
Initial fail (61456)!
Device Initial fail (61456)!
RTL9300# # 
>>> rtk i2c init hw 0 0 0 8 80 0 0
rtk i2c init hw 0 0 0 8 80 0 0
Initial fail (61456)!
Device Initial fail (61456)!
RTL9300# # 
>>> rtk network on
rtk network on
Enable network
RTCORE Driver Module Initialize
  IOAL init
  Hardware-profile probe (RTL9303_8XGE_V1)
  Hardware-profile init
  GPIO probe (unit 0): (found)
  GPIO Init
  SPI init (unit 0) 
  I2C probe (unit 0)
  I2C init (unit 0)
  RTL8231 probe (unit 0): (found)
  RTL8231 init (unit 0)
  NIC probe (unit 0)
  Loader RTNIC Driver Module Initialize
  IOAL init
RTK Driver Module Initialize
  MAC probe (unit 0)
    Chip 9303 (found)
  MAC init (unit 0)
  SMI protocol probe (unit 0)
  PHY probe (unit 0)
  Chip Construct (unit 0)
    Chip Construct
    Disable PHY Polling
    PHY Reset
    MAC Construct
    Turn Off Serdes
    Serdes Construct
    PHY Construct
    Turn On Serdes
    Mac_Polling_PHY Config
    Enable PHY Polling
    Misc
  PHY init (unit 0)
  Mgmt_dev init (unit 0) 
Please wait for PHY init-time ...

RTL9300# # 
>>> rtk i2c read 0 0
rtk i2c read 0 0
FAIL (61455)
RTL9300# # 
>>> rtk i2c read 0 1
rtk i2c read 0 1
FAIL (61455)
RTL9300# # 
>>> rtk i2c read 0 2
rtk i2c read 0 2
FAIL (61455)
RTL9300# # 
>>> rtk i2c init hw 0 0 0 8 80 0 0
rtk i2c init hw 0 0 0 8 80 0 0
 r9300_i2c_init had already been initialized!
RTL9300# # 
>>> rtk i2c read 0 0
rtk i2c read 0 0
data[0] 0x3 / data[1] 0x4
RTL9300# # 
>>> rtk i2c read 0 0
rtk i2c read 0 0
data[0] 0x3 / data[1] 0x4
RTL9300# # 
>>> rtk i2c read 0 2
rtk i2c read 0 2
data[0] 0x7 / data[1] 0x20
RTL9300# # 
>>> rtk i2c read 0 4
rtk i2c read 0 4
data[0] 0x0 / data[1] 0x0
RTL9300# # 
>>> rtk i2c read 0 6
rtk i2c read 0 6
data[0] 0x0 / data[1] 0x0
RTL9300# # 
>>> rtk i2c read 0 8
rtk i2c read 0 8
data[0] 0x0 / data[1] 0x0
RTL9300# # 
>>> rtk i2c read 0 10
rtk i2c read 0 10
data[0] 0x0 / data[1] 0x6
RTL9300# # 
>>> rtk i2c read 0 12
rtk i2c read 0 12
data[0] 0x67 / data[1] 0x0
RTL9300# # 
>>> rtk i2c read 0 14
rtk i2c read 0 14
data[0] 0xa / data[1] 0x64
RTL9300# # 
>>> rtk i2c read 0 16
rtk i2c read 0 16
data[0] 0x0 / data[1] 0x0
RTL9300# # 
>>> rtk i2c read 0 18
rtk i2c read 0 18
data[0] 0x0 / data[1] 0x0
RTL9300# # 
>>> rtk i2c read 0 20
rtk i2c read 0 20
data[0] 0x46 / data[1] 0x69
RTL9300# # 
>>> rtk i2c read 0 22
rtk i2c read 0 22
data[0] 0x62 / data[1] 0x65
RTL9300# # 
>>> rtk i2c read 0 24
rtk i2c read 0 24
data[0] 0x72 / data[1] 0x4a
RTL9300# # 
>>> rtk i2c read 0 26
rtk i2c read 0 26
data[0] 0x50 / data[1] 0x20
RTL9300# # 
>>> rtk i2c read 0 28
rtk i2c read 0 28
data[0] 0x20 / data[1] 0x20
RTL9300# # 
>>> rtk i2c read 0 30
rtk i2c read 0 30
data[0] 0x20 / data[1] 0x20
RTL9300# # 
>>> rtk i2c read 0 32
rtk i2c read 0 32
data[0] 0x20 / data[1] 0x20
RTL9300# # 
>>> rtk i2c read 0 34
rtk i2c read 0 34
data[0] 0x20 / data[1] 0x20
RTL9300# # 
>>> rtk i2c read 0 36
rtk i2c read 0 36
data[0] 0x0 / data[1] 0x0
RTL9300# # 
>>> rtk i2c read 0 38
rtk i2c read 0 38
data[0] 0x0 / data[1] 0x0
RTL9300# # 
>>> rtk i2c read 0 40
rtk i2c read 0 40
data[0] 0x53 / data[1] 0x46
RTL9300# # 
>>> rtk i2c read 0 42
rtk i2c read 0 42
data[0] 0x50 / data[1] 0x2d
RTL9300# # 
>>> rtk i2c read 0 44
rtk i2c read 0 44
data[0] 0x31 / data[1] 0x30
RTL9300# # 
>>> rtk i2c read 0 46
rtk i2c read 0 46
data[0] 0x47 / data[1] 0x42
RTL9300# # 
>>> rtk i2c read 0 48
rtk i2c read 0 48
data[0] 0x58 / data[1] 0x32
RTL9300# # 
>>> rtk i2c read 0 50
rtk i2c read 0 50
data[0] 0x33 / data[1] 0x2d
RTL9300# # 
>>> rtk i2c read 0 52
rtk i2c read 0 52
data[0] 0x31 / data[1] 0x30
RTL9300# # 
>>> rtk i2c read 0 54
rtk i2c read 0 54
data[0] 0x20 / data[1] 0x20
RTL9300# # 
>>> rtk i2c read 0 56
rtk i2c read 0 56
data[0] 0x20 / data[1] 0x20
RTL9300# # 
>>> rtk i2c read 0 58
rtk i2c read 0 58
data[0] 0x20 / data[1] 0x20
RTL9300# # 
>>> rtk i2c read 0 60
rtk i2c read 0 60
data[0] 0x4 / data[1] 0xf6
RTL9300# # 
>>> rtk i2c read 0 62
rtk i2c read 0 62
data[0] 0x0 / data[1] 0xb0
RTL9300# # 
>>> rtk i2c read 0 64
rtk i2c read 0 64
data[0] 0x0 / data[1] 0x1a
RTL9300# # 
>>> rtk i2c read 0 66
rtk i2c read 0 66
data[0] 0xe / data[1] 0x58
RTL9300# # 
>>> rtk i2c read 0 68
rtk i2c read 0 68
data[0] [REDACTED_SERIAL] / data[1] [REDACTED_SERIAL]
RTL9300# # 
>>> rtk i2c read 0 70
rtk i2c read 0 70
data[0] [REDACTED_SERIAL] / data[1] [REDACTED_SERIAL]
RTL9300# # 
>>> rtk i2c read 0 72
rtk i2c read 0 72
data[0] [REDACTED_SERIAL] / data[1] [REDACTED_SERIAL]
RTL9300# # 
>>> rtk i2c read 0 74
rtk i2c read 0 74
data[0] [REDACTED_SERIAL] / data[1] [REDACTED_SERIAL]
RTL9300# # 
>>> rtk i2c read 0 76
rtk i2c read 0 76
data[0] [REDACTED_SERIAL] / data[1] [REDACTED_SERIAL]
RTL9300# # 
>>> rtk i2c read 0 78
rtk i2c read 0 78
data[0] [REDACTED_SERIAL] / data[1] [REDACTED_SERIAL]
RTL9300# # 
>>> rtk i2c read 0 80
rtk i2c read 0 80
data[0] [REDACTED_SERIAL] / data[1] [REDACTED_SERIAL]
RTL9300# # 
>>> rtk i2c read 0 82
rtk i2c read 0 82
data[0] [REDACTED_SERIAL] / data[1] [REDACTED_SERIAL]
RTL9300# # 
>>> rtk i2c read 0 84
rtk i2c read 0 84
data[0] 0x32 / data[1] 0x36
RTL9300# # 
>>> rtk i2c read 0 86
rtk i2c read 0 86
data[0] 0x31 / data[1] 0x30
RTL9300# # 
>>> rtk i2c read 0 88
rtk i2c read 0 88
data[0] 0x30 / data[1] 0x39
RTL9300# # 
>>> rtk i2c read 0 90
rtk i2c read 0 90
data[0] 0x20 / data[1] 0x20
RTL9300# # 
>>> rtk i2c read 0 92
rtk i2c read 0 92
data[0] 0x68 / data[1] 0xf0
RTL9300# # 
>>> rtk i2c read 0 94
rtk i2c read 0 94
data[0] 0x3 / data[1] 0x4e
RTL9300# # 
>>> rtk i2c init hw 1 0 0 8 81 0 0
rtk i2c init hw 1 0 0 8 81 0 0
 r9300_i2c_init had already been initialized!
RTL9300# # 
>>> rtk i2c read 1 96
rtk i2c read 1 96
data[0] 0x2f / data[1] 0xd9
RTL9300# # 
>>> rtk i2c read 1 98
rtk i2c read 1 98
data[0] 0x82 / data[1] 0x81
RTL9300# # 
>>> rtk i2c read 1 100
rtk i2c read 1 100
data[0] 0x4a / data[1] 0xb5
RTL9300# # 
>>> rtk i2c read 1 102
rtk i2c read 1 102
data[0] 0x1a / data[1] 0x29
RTL9300# # 
>>> rtk i2c read 1 104
rtk i2c read 1 104
data[0] 0x20 / data[1] 0xce
RTL9300# # 
>>> rtk i2c read 1 110
rtk i2c read 1 110
data[0] 0x8 / data[1] 0xff
RTL9300# # 
>>> rtk i2c read 1 112
rtk i2c read 1 112
data[0] 0x0 / data[1] 0x0
RTL9300# # 
>>> rtk i2c read 1 116
rtk i2c read 1 116
data[0] 0x0 / data[1] 0x0
RTL9300# # 
>>> md.l 0xbb00cb10 1
md.l 0xbb00cb10 1
bb00cb10: 00000000    ....
RTL9300# # 
>>> md.l 0xbb000194 1
md.l 0xbb000194 1
bb000194: 007df7df    .}..
RTL9300# # 
>>> md.l 0xbb00ca1c 1
md.l 0xbb00ca1c 1
bb00ca1c: 00000194    ....
RTL9300# # 
>>> rtk phyreg get 0 0 1
rtk phyreg get 0 0 1
Get Unit 0 Port 00 page 00 reg 01    Failed!! 0xf00e 
RTL9300# # 
>>> md.l 0xbb00cb10 1
md.l 0xbb00cb10 1
bb00cb10: 10000001    ....
RTL9300# # 
>>> md.l 0xbb00cb18 1
md.l 0xbb00cb18 1
bb00cb18: 00000004    ....
RTL9300# # 
>>> md.l 0xbb00cb28 1
md.l 0xbb00cb28 1
bb00cb28: 1f110101    ....
RTL9300# # 
>>> md.l 0xbb00cb10 1
md.l 0xbb00cb10 1
bb00cb10: 10000001    ....
RTL9300# # 
>>> md.l 0xbb000194 1
md.l 0xbb000194 1
bb000194: 007df7df    .}..
RTL9300# # 
>>> rtk i2c read 1 110
rtk i2c read 1 110
data[0] 0x8 / data[1] 0xff
RTL9300# # 
>>> md.l 0xbb00cb10 1
md.l 0xbb00cb10 1
bb00cb10: 10000001    ....
RTL9300# # 
>>> md.l 0xbb00cb18 1
md.l 0xbb00cb18 1
bb00cb18: 00000004    ....
RTL9300# # 
```

### C. 抽出した識別 EEPROM（96バイト、16進表示）

調査時に作成した `sfp-port1-a0.bin` を16進表示にした。オフセットと列番号は16進数である。個体シリアル領域のみマスクしている。

```text
offset  00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f
0000    03 04 07 20 00 00 00 00 00 00 00 06 67 00 0a 64
0010    00 00 00 00 46 69 62 65 72 4a 50 20 20 20 20 20
0020    20 20 20 20 00 00 00 00 53 46 50 2d 31 30 47 42
0030    58 32 33 2d 31 30 20 20 20 20 20 20 04 f6 00 b0
0040    00 1a 0e 58 XX XX XX XX XX XX XX XX XX XX XX XX
0050    XX XX XX XX 32 36 31 30 30 39 20 20 68 f0 03 4e
```

### D. 調査に使用したシリアル操作スクリプト（最終版）

調査時に作成した `serial_query.py` の最終版をそのまま収録する。スクリプトは引数で指定されたコマンドを実行するため、初期化操作も実行できる。作業用の `work` ディレクトリーを使用し、ログを追記する。このスクリプトは受信内容をそのままログに保存する。新しいログを公開するときも、事前に個体情報をマスクする。

```python
import os, sys, signal, select, time, termios, subprocess
pids = subprocess.run(["pgrep", "-x", "minicom"], capture_output=True, text=True).stdout.split()
pid = int(pids[0]) if len(pids) == 1 else None
if len(pids) > 1:
    raise SystemExit("Multiple minicom processes; refusing ambiguous access")
fd = None
if pid is not None:
    os.kill(pid, signal.SIGSTOP)
try:
    fd = os.open('/dev/ttyUSB0', os.O_RDWR | os.O_NOCTTY | os.O_NONBLOCK)
    attrs = termios.tcgetattr(fd)
    print('Serial speed:', attrs[4], attrs[5], flush=True)
    for command in sys.argv[1:]:
        print('\n>>> ' + repr(command), flush=True)
        os.write(fd, (command + '\r').encode())
        deadline = time.monotonic() + (45 if command == "rtk network on" else 8)
        received = b''
        while time.monotonic() < deadline:
            if select.select([fd], [], [], 0.25)[0]:
                chunk = os.read(fd, 65536)
                received += chunk
                print(chunk.decode(errors='replace'), end='', flush=True)
                if b'RTL9300# #' in received:
                    break
        with open('work/serial-transcript.log', 'ab') as log:
            log.write(('\n>>> ' + command + '\n').encode() + received)
        if b'RTL9300# #' not in received:
            raise SystemExit('No U-Boot prompt returned; stopping without further commands')
finally:
    if fd is not None:
        os.close(fd)
    if pid is not None:
        os.kill(pid, signal.SIGCONT)
```

## 参照資料（既存の外部公開資料）

- [OpenWrt：SKS8310-8X の追加とポート対応](https://lists.infradead.org/pipermail/lede-commits/2025-August/026746.html)
- [OpenWrt：SKS8310-8X の RX_LOS / TX_DISABLE GPIO 割り当て修正](https://lists.infradead.org/pipermail/lede-commits/2025-October/027670.html)。古い機種定義では両信号が逆になっていたため、修正後を参照した。
- [Realtek SDK：コマンド引数処理・ヘルプ](https://github.com/reyalo/Realtek/blob/31ad7755468e4b72cd9473b4a545513de8288f47/sources/rtk-dms1250/system/uboot/cmd/uboot_cmd.c)
- [Realtek SDK：GPIO・I²C・ネットワーク初期化の実装](https://github.com/reyalo/Realtek/blob/31ad7755468e4b72cd9473b4a545513de8288f47/sources/rtk-dms1250/system/uboot/cmd/uboot_func.c)
- [Realtek SDK：エラー番号](https://github.com/reyalo/Realtek/blob/31ad7755468e4b72cd9473b4a545513de8288f47/sources/rtk-dms1250/system/include/common/error.h)
- [OpenWrt：RTL930x のリンク・速度・SerDes 定義](https://github.com/openwrt/openwrt/blob/5799d2da76998be50432b0a3325b42713b4b619b/target/linux/realtek/files-6.18/drivers/net/pcs/pcs-rtl-otto.c)
- [Linux：SFP 診断情報と状態ビットの定義](https://github.com/torvalds/linux/blob/cbbddb6b08cc9900a87cccaa9b22bbe06aa4f50a/include/linux/sfp.h)

公開 SDK は他製品向けの参照コードである。本機の動作については、このコードからの推測と実機で確認した事実を区別して記載した。
