# gdb

## 特定の関数だけdisas
```
gdb -batch -ex 'file src/LIUMOS.ELF' -ex 'disas TimerHandler'
```
