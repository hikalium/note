# gdb

## 特定の関数だけdisas
```
gdb -batch -ex 'file src/LIUMOS.ELF' -ex 'disas TimerHandler'
```

```
apt install gdb-multiarch
gdb --nx
gdb -ex 'target remote localhost:1192' src/LIUMOS.ELF

```
