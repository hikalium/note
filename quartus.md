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

## Ubuntu 16.04.2 LTS
```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install kubuntu-desktop
# これで再起動すればGUIになってる
何も考えずに展開してsetup.shを叩けばOK

```
