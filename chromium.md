## Build Chromium 
```
## update repos
git rebase-update
gclient sync
gn gen out/Default
gn args out/Default
ninja -C out/Default chrome
## test
ninja -C out/Default blink_tests
(mac) strip out/Default/Content\ Shell.app/Contents/MacOS/Content\ Shell
python third_party/WebKit/Tools/Scripts/run-webkit-tests -t Default 
```

### Common
- https://chromium.googlesource.com/chromium/src/+/master/tools/gn/docs/quick_start.md
  - `gn args out/<any>`とすると、エディタが立ち上がってビルドパラメータをいじれる

### OSX
- 手順に従えばふつうにできた
  - https://chromium.googlesource.com/chromium/src/+/master/docs/mac_build_instructions.md
- content_shellをstripしようとしたけど場所がわからなかった
  - `strip out/Default/Content\ Shell.app/Contents/MacOS/Content\ Shell`

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
    
### Ubuntu 16.04 LTS

- 本体のビルドは正常にできた。起動も問題ない。
- テストが走らない。

  - Workaroundを発見 -> https://codereview.chromium.org/2740483002/

  - これも関係ある? -> https://chromium.googlesource.com/chromium/src/+/lkcr/docs/layout_tests_linux.md#pixel-tests

    - `37: trusty`を選択(14.04相当)
