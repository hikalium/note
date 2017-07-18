# Quartu Prime 関連
## CentOS7

### 共通
```
sudo yum update
sudo yum groupinstall "Development tools"
reboot
```

### Guest Addons(Virtual Boxのみ)
```
# CD挿入
sudo mount /dev/cdrom /mnt
cd /mnt
sudo ./VBoxLinuxAdditions.run

```
### Quartusに必要なパッケージ
```
sudo yum groupinstall "KDE Plasma Workspaces"
sudo systemctl set-default graphical.target
sudo reboot
sudo yum install libpng12 glibc.i686
sudo yum install libX11-devel.i686 libXext-devel.i686 libXft-devel.i686 ncurses-libs.i686
```

## Ubuntu 16.04.2 LTS
```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install kubuntu-desktop
# これで再起動すればGUIになってる
何も考えずに展開してsetup.shを叩けばOK

```

## Windows 8.1 Pro
- tarを解凍する際、Lhaplusではうまく行かず。7Zipではうまく解凍できた。
- 解凍したフォルダの`setup.bat`を実行すればよい、インストーラの設定はすべてデフォルト値でOK.
- USB Blasterが認識されていなかったので、手動でドライバを更新（デバイスマネージャより）
  - `C:\intelFPGA_lite\17.0\quartus\drivers\usb-blaster`をドライバのパスとして指定すればOK
- するとUSB Blaster自体は認識されるようになった。
- その後すぐにProgrammerを走らせようとすると`Unable to scan device chain`となった。
- 再起動してもProgrammerを起動しているとしばしばBSoDに出会うように。つらい。
- 以下のフォーラムを参考に、古いProgrammer(16.1)をインストールしてみた
  - https://alteraforum.com/forum/showthread.php?t=54887
- 解決せず。ドライバも古くする必要があるのではないかと考え、古いドライバを削除（ここでまたBSoD）して新しいドライバをインストール。
- うまくいっていない模様

