## Mac OSX
### スクリーンショットを特定フォルダに保存する
```
defaults write com.apple.screencapture location ~/Desktop/SS
```

### OS, CPU情報の取得
```
sw_vers
sysctl -a | grep machdep.cpu.brand_string
```



## MacOSXの環境構築
- Chromeを入れる(Chromiumにする？)
- dotfilesを入れる
- ターミナルのカラースキームを設定
  - SourceCodeProを入れる
- Homebrewを入れる
  - tmux
  - reattach-to-user-namespace
- 各種サーバに公開鍵を登録
- texを入れる（BasicTexでいいかな）
- スクリーンショットの保存フォルダを変更
- Wireshark
