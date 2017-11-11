## AVTOKYOでもらったボードの話

- 以下の互換ボードっぽい
  - http://wiki.stm32duino.com/index.php?title=Blue_Pill
- 会場で一緒にST-LINK V2 の互換品も買った
- `brew stlink` でツールを入れる

### 解析
起動時に一瞬だけフラッシュにアクセスできる?その瞬間をねらって `st-util`する。
```
$ st-util
st-util 1.4.0
2017-11-11T23:51:37 INFO src/usb.c: -- exit_dfu_mode
2017-11-11T23:51:37 INFO src/common.c: Loading device parameters....
2017-11-11T23:51:37 INFO src/common.c: Device connected is: F1 Medium-density device, id 0x20036410
2017-11-11T23:51:37 INFO src/common.c: SRAM size: 0x5000 bytes (20 KiB), Flash: 0x10000 bytes (64 KiB) in pages of 1024 bytes
2017-11-11T23:51:37 INFO src/gdbserver/gdb-server.c: Chip ID is 00000410, Core ID is  1ba01477.
2017-11-11T23:51:37 INFO src/gdbserver/gdb-server.c: Listening at *:4242...
```
