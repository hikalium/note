# gdb

## 特定の関数だけdisas
```
gdb -batch -ex 'file src/LIUMOS.ELF' -ex 'disas TimerHandler'
```

```
apt install gdb-multiarch
gdb --nx
gdb -ex 'target remote localhost:1192' src/LIUMOS.ELF

gdb minimal_linux_elf_00.bin \
  -ex 'set disable-randomization off' \
  -ex 'starti' \
  -ex 'set pagination off' \
  -ex 'info target' \
  -ex 'set confirm off' \
  -ex 'quit' \
| grep '.text'
```
