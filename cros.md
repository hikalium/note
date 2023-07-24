# ChromiumOS hacking note (from publically available information)

## EC / Servo commands

### Servo

There are three channels:
- Servo EC Shell
- DUT UART
- Atmega UART

https://source.chromium.org/chromiumos/chromiumos/codesearch/+/main:src/platform/release-firmware/fpmcu-bloonchipper/board/servo_v4p1/board.c;l=476-478;drc=1d44b153dcae4320bd3d815d06d2f8082e9a48eb

#### Servo EC Shell
- macaddr
- ioexget
- ioexset

#### Servo ioext
- ioexget
- ioexset

```
# reset ETH
ioexset EN_PP3300_ETH 0
ioexset EN_PP3300_ETH 1
```
https://source.chromium.org/chromiumos/chromiumos/codesearch/+/main:src/platform/release-firmware/fpmcu-nami/board/servo_v4p1/ioexpanders.h;l=159-167;drc=71b2ef709dcb14260f5fdaa3ab4ced005a29fb46



## Install Authy Desktop for Linux!

![Screenshot 2023-04-30 22 36 23](https://user-images.githubusercontent.com/10512779/235356396-30449e9c-971a-47ce-9913-599b6a78439f.png)

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
# xhost + can also be used (as Twilio suport suggested)

sommelier -X authy
# or
xhost + && authy

# enjoy!
# verified on: 15428.0.0 (Official Build) dev-channel brya
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
