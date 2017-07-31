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
