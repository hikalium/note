# ChromiumOS hacking note (from publically available information)

## download official recovery images
source: https://groups.google.com/a/chromium.org/g/chromium-os-discuss/c/OcB4zaIWwok
```
wget https://dl.google.com/dl/edgedl/chromeos/recovery/recovery.conf
export IMAGE_ZIP_URL=`cat recovery.conf | grep -A 8 AKEMI-AOKF | grep -E ^url= | cut -d = -f 2-`
echo $IMAGE_ZIP_URL
wget $IMAGE_ZIP_URL
```
