# gentoo commands

## daily update
https://wiki.gentoo.org/wiki/Upgrading_Gentoo
```
sudo emaint --auto sync
sudo emerge --ask --verbose --update --deep --newuse @world
```

## misc

```
emerge --sync # sync to latest
emerge-webrsync # sync to snapshot

eselect news list
eselect news read
man news.eselect
eselect profile list

eselect locale list

ls /var/cache/distfiles/
```

```
dev-libs/protobuf
dev-util/wayland-scanner
gst-plugins-base
gui-libs/wlroots
info
lsusb
net-misc/ntp
ntpd
opensc
protoc
rtcwake
rust
rustup
scdaemon
sys-apps/lssbus
sys-apps/usbutils
sys-fs/dosfstools
usbutils
wayland-protocols
xdg-shell
```
