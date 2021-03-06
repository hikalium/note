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

```
enable_nacl=false
is_debug=false
remove_webcore_debug_symbols=true
```

## Links
- [Issues](https://bugs.chromium.org/p/chromium/issues/list)
  - [Good for first bug, Linux & Mac, Available](https://bugs.chromium.org/p/chromium/issues/list?can=2&q=Hotlist%3DGoodFirstBug+status%3AAvailable+OS%3DLinux%2CMac&colspec=ID+Pri+M+Stars+ReleaseBlock+Component+Status+Owner+Summary+OS+Modified&x=m&y=releaseblock&cells=ids)
- https://build.chromium.org/p/chromium/console
  - 各コミットのテスト状況が見れる？
- https://chopsdash.appspot.com/
  - Chrome開発関連の各種ページの稼働状況が見れる。

## Memo

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
    
### Ubuntu 14.04.5 LTS
- `apt-get install g++-4.8-multilib` をしないと `./build/install-build-deps.sh` が通らない。
### Ubuntu 16.04 LTS

- 本体のビルドは正常にできた。起動も問題ない。
- テストが走らない。

  - Workaroundを発見 -> https://codereview.chromium.org/2740483002/

  - これも関係ある? -> https://chromium.googlesource.com/chromium/src/+/lkcr/docs/layout_tests_linux.md#pixel-tests

    - `37: trusty`を選択(14.04相当)
