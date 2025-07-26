# Linux Environment on ChromeOS (crostini)

## (Unofficial) App compat report

### Blender

```
$ cat /opt/google/cros-containers/etc/lsb-release | grep _DESC
CHROMEOS_RELEASE_DESCRIPTION=16266.0.0 (Official Build) dev-channel tatl test

# Blender 4.5 LTS
$ wget https://download.blender.org/release/Blender4.5/blender-4.5.0-linux-x64.tar.xz
# -> can be launched but could not be interacted (no click recognized)

# Blender 4.2 LTS
$ wget https://download.blender.org/release/Blender4.2/blender-4.2.12-linux-x64.tar.xz
# -> can be launched but could not be interacted (no click recognized)


```

```

```

```
$ cat /opt/google/cros-containers/etc/lsb-release
CHROMEOS_RELEASE_NAME=Chrome OS
CHROMEOS_AUSERVER=https://tools.google.com/service/update2
CHROMEOS_DEVSERVER=
CHROMEOS_RELEASE_BUILDER_PATH=tatl-release/R137-16266.0.0
CHROMEOS_RELEASE_KEYSET=devkeys
CHROMEOS_RELEASE_TRACK=dev-channel
CHROMEOS_RELEASE_BUILD_TYPE=Official Build
CHROMEOS_RELEASE_DESCRIPTION=16266.0.0 (Official Build) dev-channel tatl test
CHROMEOS_RELEASE_BOARD=tatl
CHROMEOS_RELEASE_BRANCH_NUMBER=0
CHROMEOS_RELEASE_BUILD_NUMBER=16266
CHROMEOS_RELEASE_CHROME_MILESTONE=137
CHROMEOS_RELEASE_PATCH_NUMBER=0
CHROMEOS_RELEASE_VERSION=16266.0.0
GOOGLE_RELEASE=16266.0.0
```

### Ultimaker_Cura-4.13.1.AppImage
- description: 3D slicer app
- result: worked
- link: https://github.com/Ultimaker/Cura/releases/tag/4.13.1
- note: Latest version will SEGV on file access somehow

```
hikalium@penguin:~$ cat /opt/google/cros-containers/etc/lsb-release 
CHROMEOS_RELEASE_NAME=Chrome OS
CHROMEOS_AUSERVER=https://tools.google.com/service/update2
CHROMEOS_DEVSERVER=
CHROMEOS_RELEASE_BUILDER_PATH=tatl-release/R115-15474.83.0
CHROMEOS_RELEASE_KEYSET=devkeys
CHROMEOS_RELEASE_TRACK=dev-channel
CHROMEOS_RELEASE_BUILD_TYPE=Official Build
CHROMEOS_RELEASE_DESCRIPTION=15474.83.0 (Official Build) dev-channel tatl test
CHROMEOS_RELEASE_BOARD=tatl
CHROMEOS_RELEASE_BRANCH_NUMBER=83
CHROMEOS_RELEASE_BUILD_NUMBER=15474
CHROMEOS_RELEASE_CHROME_MILESTONE=115
CHROMEOS_RELEASE_PATCH_NUMBER=0
CHROMEOS_RELEASE_VERSION=15474.83.0
GOOGLE_RELEASE=15474.83.0
hikalium@penguin:~$ uname -a
Linux penguin 5.15.112-19404-g55fe7e355056 #1 SMP PREEMPT Tue Aug 8 19:05:00 PDT 2023 x86_64 GNU/Linux   
hikalium@penguin:~$ cat /etc/debian_version 
11.7
hikalium@penguin:~$ date
Mon 28 Aug 2023 01:39:06 PM JST
```
