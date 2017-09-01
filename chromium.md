## Build Chromium 
### Common
- https://chromium.googlesource.com/chromium/src/+/master/tools/gn/docs/quick_start.md
  - `gn args out/<any>`とすると、エディタが立ち上がってビルドパラメータをいじれる

### OSX
- 手順に従えばふつうにできた
  - https://chromium.googlesource.com/chromium/src/+/master/docs/mac_build_instructions.md
- content_shellをstripしようとしたけど場所がわからない
  - コマンドの出力結果より、以下の場所らしい
  - `/Users/hikalium/repos/chromium/src/out/Default/Content Shell.app/Contents/MacOS/Content Shell`

### Arch Linux
- Instructionの下の方に、各ディストリビューションの方法が一応書いてある。
  - https://chromium.googlesource.com/chromium/src/+/master/docs/linux_build_instructions.md
- python2を呼び出してくれなくてgclient runhooksが走らない。
  - issueは立ってる。
    - https://codereview.chromium.org/1437773002/
  - `!/usr/bin/env python`を参照しているので以下のトリックが使えた
    - https://wiki.archlinuxjp.org/index.php/Python
- いろいろライブラリ・バイナリ不足でコンパイルが落ちる
  - `libtinfo.so.5` -> https://aur.archlinux.org/packages/libtinfo5/
  - `gperf` -> `sudo pacman -Syu gperf`
  - `libnss3.so` -> `sudo pacman -Syu nss`

- 実行時にも落ちる
  - `sudo pacman -Syu libxcomposite libcups libxcursor libxss gconf atk alsa-lib libxtst libxrandr gtk3`
  - `--no-sandbox`をつけないと起動しない。
    - `linux-userns`を入れてloaderをいじったがそれでも落ちる。
