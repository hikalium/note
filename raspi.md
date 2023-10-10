# raspi

## Get GPU temp
```
$ vcgencmd measure_temp --help
temp=40.1'C
```

## Get CPU temp
```
$ cat /sys/class/thermal/thermal_zone0/type 
cpu-thermal

$ cat /sys/class/thermal/thermal_zone0/temp
39546
```

## burn SD card
```
sudo dd if=2023-05-03-raspios-bullseye-armhf-lite.img of=/dev/sdd bs=2048 status=progress
```

## ssh setup

```
$ sudo fdisk -l /dev/sdd
Disk /dev/sdd: 29.81 GiB, 32010928128 bytes, 62521344 sectors
Disk model: MassStorageClass
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x4c4e106f

Device     Boot  Start     End Sectors  Size Id Type
/dev/sdd1         8192  532479  524288  256M  c W95 FAT32 (LBA)
/dev/sdd2       532480 3842047 3309568  1.6G 83 Linux

$ TMPDIR=`mktemp -d`
$ sudo mount /dev/sdd1 ${TMPDIR}

# Set ${YOUR_PASS} here
$ PASS_HASH=`echo "${YOUR_PASS}" | openssl passwd -6 -stdin`
$ sudo bash -c "echo ${USER}:${PASS_HASH} | tee ${TMPDIR}/userconf"
# redacted
$ sudo umount ${TMPDIR}
$ sync
```
