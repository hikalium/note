# Quartu Prime 関連
## CentOS7

### Guest Addons
```
sudo yum update
sudo yum groupinstall "Development tools"
reboot

# CD挿入
sudo mount /dev/cdrom /mnt
cd /mnt
sudo ./VBoxLinuxAdditions.run

```
### Quartusに必要なパッケージ
```
sudo yum groupinstall "KDE Plasma Workspaces"
sudo systemctl set-default graphical.target
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
