# ctf memo by hikalium

## バイナリファイルからHexのダンプをつくる
```
hexdump -ve '1/1 "%.2x"' <file>
xxd -p <file>
```

## xxdでバイナリファイルを作る
```
cat attack_hex.txt | xxd -r -p > attack_hex.bin
```

## みんな大好きstrace
```
strace ./challenge_bof.elf < attack_hex.bin
```

## execveで/bin/shを呼ぶ
```
59: execve
rdi    filename
rsi    argc
rdx    envp
```

## netstatで空いてるポートチェック
```
netstat -an | grep LISTEN
```

## lsof
```
lsof -p <PID>
```
そのプロセスが開いているファイル一覧を取れる

## objdump(gobjdump)
```
-d # disasm
```

## gdb
AT&T記法（%がつくやつ）では、movは左から右に代入
```
si      step instr
disas
attach <PID>
info files
info reg 
b *<addr>
tb *<addr>
del <break point id>
x/10c $rax
x/10i $rip
x/16bx $rdi
x/16gx $rsp
p/x *(uint64_t *)$rsp
info proc mappings
```

## syscall(64bit)
戻り値はrax

```
9: mmap
    rdi     addr
    rsi     len
    rdx     prot
    r10     flags
    r8      fd
    r9      off
```
```
59 == 0x3B: execve
 rax = 0x3B
 rdi    filename
 rsi    argv
 rdx    envp
```

## pkcrack
```
pkcrack -C <encrypted_zip> -c <known_file_name> -p <known_file> -d <decrypt_out_zip>
```

## MySQL
```
"admin'-- "
"' union select * from table2 where 'a' = 'a"
```
