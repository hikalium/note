# QEMU 周辺機器を書くときの罠 5 点

PCI 周辺機器を QEMU に新規実装したとき (chromeos-6.12 i915 PXP の vtag passthrough を `/dev/i915_pxp_test` 経由で end-to-end 検証するため `hw/misc/intel-mei.c` を書いた経緯) のメモ。`qemu.md` (upstream にパッチを送る側の手順) と対になる「QEMU で実機を模す側」のノート。

## 1. PCI INTx は level-triggered

罠: `pci_set_irq(s, 1)` を呼んだまま下げ忘れると、kernel ISR が無限ループ。実測 200,000+ MMIO accesses まで動作停止。

対処: 「line state = (IS && IE)」を関数化して、IS/IE が変わるすべての点で呼ぶ。

```c
static void dev_irq_eval(DevState *s)
{
    int level = ((s->int_status & s->int_enable_mask) != 0) ? 1 : 0;
    pci_set_irq(&s->pdev, level);
}
```

呼ぶべき場所:

- IS を立てたとき (raise)
- IE を立てた / 落としたとき (kernel CSR write の handler 内)
- IS を W1C で消したとき
- 未読データがまだあれば IS を re-latch してから `dev_irq_eval()`

参考: `hw/misc/edu.c` は MSI 前提なので edge-friendly。INTx を書くなら自分で line state を管理する必要がある。

## 2. MMIO レジスタはビット単位で access semantics を区別

罠: `me_intr_disable` は `H_IS` を strip した値で H_CSR を書く。これを「val をそのまま `s->reg` にコピー」すると、kernel が "I just want to clear IE, leave IS alone" のつもりが意図と違う動きになる。

対処: ビットを R/W、W1C、RO、W1S にグルーピングして書き込みハンドラを書く。

```c
#define DEV_RW_MASK   (BIT_RST | BIT_RDY | BIT_IG | BIT_IE)
#define DEV_W1C_MASK  (BIT_IS | BIT_D0I3_IS)
#define DEV_RO_MASK   (BIT_CBD | BIT_CBWP | BIT_CBRP)  /* host read-only */

static void dev_csr_write(DevState *s, uint32_t old, uint32_t val)
{
    /* R/W: replace */
    s->reg = (s->reg & ~DEV_RW_MASK) | (val & DEV_RW_MASK);
    /* W1C: clear bits where val has 1 */
    s->reg &= ~(val & DEV_W1C_MASK);
    /* RO: untouched (already masked out above) */
}
```

データシートは「ビット名」だけで読むと罠。**毎ビットの access semantics** まで確認するルールにする。

## 3. State transition (edge) と steady state (level) を区別

罠: 「`H_RST=1` だったらリセット」と書くと、`H_RST` を立てたまま H_CSR を別目的で書かれる (e.g. `me_intr_clear`) 度にフルリセットが走り、CB のデータが消える。

対処: edge detect。

```c
bool rising  = !(old & BIT_RST) &&  (val & BIT_RST);
bool falling =  (old & BIT_RST) && !(val & BIT_RST);

if (rising)   do_reset_assert(s);
if (falling)  do_reset_release(s);
/* NEVER: if (val & BIT_RST) do_reset() — fires every steady-state write */
```

HW のリセットボタンは「押した瞬間」が事象。状態ではなく **遷移** に反応する。

## 4. 仮想 HW の "ゼロ遅延" は kernel 側の latent race を曝す

罠: 同期 echo を返す合成 client を作ったら、kernel の `read(2)` が pending CB を queue する前に応答が届き、`mei_cl_irq_read_msg` の `cl->rd_pending` 空チェックで discard。実機の CSE FW は応答に ms オーダーかかっており、その latency が race window を埋めていた。

```
時刻 t:   write(2) → kernel が H_CB_WW に書く → QEMU が echo を queue + IRQ
時刻 t+ε: kernel ISR → cl->rd_pending 探す → 空! → discard
時刻 t+δ: userspace が read(2) → CB queue → wait → 永遠に来ない
```

対処 3 通り、状況で使い分け:

| 方法 | いつ |
|---|---|
| 仮想デバイスを `fixed_address` クライアントに | kernel 内で on-demand CB 確保 + buffering する path がある (MEI ならこれ) |
| QEMU 内で `QEMUTimer` で 10-50ms 噛ます | dynamic client セマンティクスを保ちたい時 |
| `qemu_bh_schedule` | BH は次の main loop iteration で fire、guest CPU から見ると非常に近い時刻。実 HW の latency simulation には足りない |

気づき: 仮想化でしか発火しない race は、しばしば実 HW でも latent。実 HW の latency が偶然救っているだけ。発見したらバグレポート + 別途記録。

## 5. Kernel test fixture は in-tree `#include` パターンで static 関数にアクセス

罠: 検証したい関数が `static`。`EXPORT_SYMBOL_GPL` を本番に足すと signature 汚染。別 module から `kallsyms_lookup_name` は 5.7+ で禁止。

対処: i915 selftests と同じ idiom — 本番 `.c` の末尾で test を `#include` する。

```c
/* drivers/foo/foo_main.c の末尾 */
#if IS_ENABLED(CONFIG_FOO_TEST_DEV)
#include "selftests/foo_test_dev.c"
#endif
```

```c
/* drivers/foo/selftests/foo_test_dev.c */
int foo_test_dev_module_init(void)
{
    /* register a miscdev */
    /* build fake state on heap, call static internals directly */
}
```

```
# drivers/foo/Kconfig.debug
config FOO_TEST_DEV
    bool "Enable foo test miscdev"
    depends on FOO
```

```c
/* drivers/foo/foo_module.c の init_funcs[] */
#include "selftests/foo_test_dev.h"
...
{ .init = foo_test_dev_module_init, .exit = foo_test_dev_module_exit },
```

注意点:

- kbuild の include path は `-I$(src)` で **module root 相対**。サブディレクトリの `.c` から書くときも `pxp/selftests/foo.h` のように module root から書く
- `menu "..." depends on EXPERT` 配下に Kconfig を置くと `make olddefconfig` が **silently drop** する。`grep CONFIG_FOO_TEST_DEV .config` で消えてたら EXPERT を疑う
- `drm_dbg(NULL, ...)` は safe (`drm_dev_dbg` が NULL check) → fake `struct drm_device` を作らなくても printk は崩れない
- 本番構造体はスタックでなく `kzalloc` で確保。zero-init で init path 多数の field が「未触」になり minimal setup で済む
- 本番関数は同一 TU から呼べる → static 関数も自由に試験可

参考: `drivers/gpu/drm/i915/i915_gem.c` の末尾 `#include "selftests/mock_gem_device.c"` 系。

## 参考リンク

- https://www.qemu.org/docs/master/devel/index.html — QEMU developer docs
- https://www.qemu.org/docs/master/devel/qom.html — QOM (QEMU Object Model)
- https://wiki.qemu.org/Documentation/Building — build
- https://github.com/qemu/qemu/blob/master/hw/misc/edu.c — PCI device 雛形 (MSI 前提)
- https://docs.kernel.org/driver-api/mei/mei-client-bus.html — MEI bus 概念
- https://www.kernel.org/doc/html/latest/dev-tools/kunit/index.html — KUnit (selftest と並ぶ in-tree test 機構)
