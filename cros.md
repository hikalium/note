# ChromiumOS hacking note (from publically available information)

# Install Authy Desktop Linux
```
# on Crostini (the Linux environment on ChromeOS)
sudo apt install snapd squashfuse
# squashfuse is needed to fix the following error:
# $ sudo snap install authy
# error: system does not fully support snapd: cannot mount squashfs image using "squashfs": mount:
#        /tmp/sanity-mountpoint-294195823: mount failed: Operation not permitted.
sudo snap install authy
# in my case, the first attempt is failed with:
# error: cannot perform the following tasks:
# - Setup snap "snapd" (18933) security profiles (cannot run udev triggers: exit status 1
# udev output:
# Failed to write 'change' to '/sys/devices/LNXSYSTM:00/uevent': Permission denied
# but running exactly the same command somehow solved the issue
sudo snap install authy
# 2023-04-30T22:23:56+09:00 INFO Waiting for automatic snapd restart...
# authy 2.3.0 from Twilio Authy installed
# Now, let's start it with sommelier (pre-installed wayland compositor with X forwarding support)
# c.f. https://chromium.googlesource.com/chromiumos/platform2/+/HEAD/vm_tools/sommelier/README.md
sommelier -X authy
```

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
