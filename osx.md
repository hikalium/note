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
- dotfilesを入れる
- SourceCodeProを入れる
- texを入れる（BasicTexでいいかな）
- スクリーンショットの保存フォルダを変更
