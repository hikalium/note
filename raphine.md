# Undocumented Raph_Kernel

## init スクリプト
- `source/arch/hw/x86/init_script/default`がデフォルトスクリプトだが、これを書き換えるのはよろしくない。
- `source/arch/hw/x86/init_script/<git branch name>`を作成しておけばこれが起動直後に実行されるのでこちらを利用すべき。

## makeコマンド
- `make`
- `make run`
- `make vnc`
- `make run_pxeserver`

## カーネルパニックのスタックトレース
- アドレスのLSB32bit部分の16進数を0xをつけずに`./debug.sh`に渡してあげると発生場所がわかる（かもしれない）。
```
$ ./debug.sh 80111f8f
Shell::Register(char const*, void (*)(int, char const**))
/vagrant/source/kernel/shell.cc:52
Shared connection to 127.0.0.1 closed.
```
