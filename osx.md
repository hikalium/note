## Mac OSX

### Reboot without preboot auth when filevault is set

```
sudo fdesetup authrestart
```

### Install basic tools
```
xcode-select --install
# ウィンドウが開くのでインストールを選ぶ
```

### スクリーンショットを特定フォルダに保存する
```
defaults write com.apple.screencapture location ~/Desktop/SS
```

### OS, CPU情報の取得
```
sw_vers
sysctl -a | grep machdep.cpu.brand_string
```

### NetBIOS名からIPを調べる
```
smbutil lookup <NetBIOS name>
```

## ホスト名を設定する
- ComputerName
  - The user-friendly name for the system
- HostName
  - The name associated with hostname(1) and gethostname(3)
- LocalHostName
  - The local (Bonjour) host name

```
sudo scutil --set ComputerName txx.z01.hikalium.com
sudo scutil --set HostName txx.z01.hikalium.com
sudo scutil --set LocalHostName txx
```


### rmをゴミ箱移動に変更する
https://qiita.com/icedpasta1832/items/695590b67440347d5fde
```
brew install rmtrash
alias rm='rmtrash'
```


## MacOSXの環境構築
- Chromeを入れる(Chromiumにする？)
- dotfilesを入れる
- ターミナルのカラースキームを設定
  - SourceCodeProを入れる
- Homebrewを入れる
  - tmux
  - reattach-to-user-namespace
- [brew/cask](https://caskroom.github.io/)
  - brew tap caskroom/cask
  - brew cask install basictex
  - http://web-salad.hateblo.jp/entry/2014/02/03/000000
  - https://texwiki.texjp.org/?TeX%20Live%2FMac
- 各種サーバに公開鍵を登録
- texを入れる（BasicTexでいいかな）
- スクリーンショットの保存フォルダを変更
- Wireshark
