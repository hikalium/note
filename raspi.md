# raspi

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

$ ls ${TMPDIR}
bcm2708-rpi-b.dtb       bcm2710-rpi-2-b.dtb       bcm2711-rpi-cm4.dtb     fixup4.dat    kernel7.img       start4db.elf
bcm2708-rpi-b-plus.dtb  bcm2710-rpi-3-b.dtb       bcm2711-rpi-cm4-io.dtb  fixup4db.dat  kernel7l.img      start4.elf
bcm2708-rpi-b-rev1.dtb  bcm2710-rpi-3-b-plus.dtb  bcm2711-rpi-cm4s.dtb    fixup4x.dat   kernel8.img       start4x.elf
bcm2708-rpi-cm.dtb      bcm2710-rpi-cm3.dtb       bootcode.bin            fixup_cd.dat  kernel.img        start_cd.elf
bcm2708-rpi-zero.dtb    bcm2710-rpi-zero-2.dtb    cmdline.txt             fixup.dat     LICENCE.broadcom  start_db.elf
bcm2708-rpi-zero-w.dtb  bcm2710-rpi-zero-2-w.dtb  config.txt              fixup_db.dat  overlays          start.elf
bcm2709-rpi-2-b.dtb     bcm2711-rpi-400.dtb       COPYING.linux           fixup_x.dat   ssh               start_x.elf
bcm2709-rpi-cm2.dtb     bcm2711-rpi-4-b.dtb       fixup4cd.dat            issue.txt     start4cd.elf

$ sudo umount ${TMPDIR}
$ sync
```
