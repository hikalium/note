# CentOS7

## VirtualBoxのインストール
```
wget http://download.virtualbox.org/virtualbox/rpm/el/virtualbox.repo
sudo mv virtualbox.repo /etc/yum.repos.d/
sudo yum install VirtualBox-5.1.x86_64
```

## ディスクの追加
```
fdisk -l
# どのデバイスか特定する
sudo fdisk <device>
# dで消す
# nで新規作成
# wで書き込み

# ファイルシステムをつくる
sudo mkfs -t ext4 <device>
# UUIDをしらべる
sudo blkid <device> 
# 自動でマウントされるように設定
vi /etc/fstab
```

## exfat
```
sudo rpm -Uvh http://li.nux.ro/download/nux/dextop/el7/x86_64/nux-dextop-release-0-5.el7.nux.noarch.rpm
sudo yum install exfat-utils
sudo yum install fuse-exfat
```
