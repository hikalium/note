# ChromiumOS hacking note (from publically available information)

## ARM Fuzzing board is using kexec
https://source.chromium.org/chromiumos/chromiumos/codesearch/+/main:src/overlays/overlay-amd64-generic-koosh/README.md;drc=a01ca05e20b74fea753182f8328382c043d6f14c

## CodeSearch
https://source.chromium.org/chromiumos

## download official recovery images
source: https://source.chromium.org/chromiumos/chromiumos/codesearch/+/main:src/third_party/coreboot/util/chromeos/crosfirmware.sh;drc=84fba38fad7fc9acb53d13b1b6edd85403e3b188
```
wget https://dl.google.com/dl/edgedl/chromeos/recovery/recovery.conf
export IMAGE_ZIP_URL=`cat recovery.conf | grep -A 8 AKEMI-AOKF | grep -E ^url= | cut -d = -f 2-`
echo $IMAGE_ZIP_URL
wget $IMAGE_ZIP_URL
```
