# ChromiumOS hacking note (from publically available information)

## random html from Crostini
```
$ /opt/google/cros-containers/bin/garcon --client --url 'data:text/html,<script>window.location.href="https://hikalium.com"</script>'
```

## open a file in a browser from Crostini
```
garcon-url-handler <path_to_file>
```

## API key

https://www.chromium.org/developers/how-tos/api-keys/#providing-keys-at-runtime

## gbb flags
```
# futility gbb --get --flash --flags 2>/dev/null
flags: 0x00000000
```

## Compatibility
### Hardware
#### Apple Pro Display XDR
- monitor_info: ProDisplayXDR : Manufacturer: APP - Product ID: AE2D - Year of Manufacture: 2019
- Worked with felwinter (ASUS Chromebook Flip CX5(CX5601), ASUS Chromebook Enterprise Flip CX5(CX5601)) @ 15474.84.0 (Official Build) stable-channel brya
- Did NOT work with duffy (ASUS Chromebox 4) @ 15474.84.0 (Official Build) stable-channel puff

## random
```
./update_kernel.sh --remote 10.10.10.45 --arch x86_64 --remote_bootargs --device sda --partition /dev/sda2 --vboot --nosyslinux

crossystem dev_default_boot=usb
crossystem dev_default_boot=disk

# IO port tweak
# https://people.redhat.com/rjones/ioport/

# hatch / akemi
# https://www.intel.com/content/dam/www/public/us/en/documents/datasheets/9-series-chipset-pch-datasheet.pdf
# 12.7.1 NMI_SC - NMI Status and Control Register
./port --read --size 1 0x61
# 52 = 0101_0010
# 36 = 0011_0110
./port --write --size 1 0x61 3 # beep on
./port --write --size 1 0x61 0 # beep off

# 12.3.1 TCW - Timer Control Word Register
# Write Only
./port --write --size 1 0x43 0xb6 # 0b1011_011_0, counter 2, counter write LSB then MSB, square wave, binary counter
./port --write --size 1 0x42 0x20   # PIT beep counter LSB
./port --write --size 1 0x42 0x00   # PIT beep counter MSB

./port --read --size 1 0x42   # PIT beep counter read


# 12.7.5 RST_CNT - Reset Control Register
./port --write --size 1 0xcf9 14    # reset the machine

# https://source.chromium.org/chromiumos/chromiumos/codesearch/+/main:src/platform/depthcharge/src/board/hatch/board.c;l=134?q=hatch%20beep&ss=chromiumos%2Fchromiumos%2Fcodesearch:src%2F
# 

```

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

via this command, we can toggle things on Servo.

https://source.chromium.org/chromiumos/chromiumos/codesearch/+/main:src/platform/release-firmware/fpmcu-nami/board/servo_v4p1/ioexpanders.h

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

```
# URLs extracted from the recovery utility
      'https://dl.google.com/dl/edgedl/chromeos/recovery/recovery2.json',
      'https://dl.google.com/dl/edgedl/chromeos/recovery/cloudready_recovery2.json'
```

## download chromeos flex latest usb stick image
- https://dl.google.com/chromeos-flex/images/latest.bin.zip
- source: https://support.google.com/chromeosflex/answer/11541904

# crosh help_advanced


```
crosh> help_advanced
Usage: arc
  [ ping [ NETWORK ] [ <ip address> | <hostname> ] |
    http [ NETWORK ] <url> |
    dns [ NETWORK ] <domain> |
    proxy <url> |
    list [ networks ] |
    stats [ sockets | traffic ]
  ]
  where NETWORK := [ wifi | eth | ethernet | cell | cellular | vpn ]

ping:           check the reachability of a host or IP address.
http:           do a GET request to an URL and print the response header.
dns:            perform a DNS lookup of a domain name.
proxy:          resolve the current proxy configuration for a given URL.
list networks:  show properties of all networks connected in Android.
stats sockets:  show TCP connect and DNS statistics by Android Apps.
stats traffic:  show traffic packet statistics by Android Apps.

If NETWORK is not specified, the default network is used.
autest [--scheduled]  
  Trigger an auto-update against a **test** update server.

  WARNING: This may update to an untested version of ChromeOS which was never
  intended for end users!

  The --scheduled option fakes a scheduled update.

authpolicy_debug <level>  
  Set authpolicy daemon debugging level.
  <level> can be 0 (quiet), 1 (taciturn), 2 (chatty), or 3 (verbose).

battery_firmware <info>  
  info   : Query battery info.

battery_test [<test length>]  
  Tests battery discharge rate for given number of seconds. Without an argument,
  defaults to 300 seconds.


  Enters a Bluetooth debugging console.
builtin_corpssh
  [Deprecated] Enable or disable CorpSSH support for the built-in gnubby.
  This is now always enabled, and the command will be removed soon.

ccd_pass
  When prompted, set or clear CCD password (use the word 'clear' to clear
  the password).

chaps_debug [start|stop|<log_level>]  
  Sets the chapsd logging level.  No arguments will start verbose logging.

connectivity   
  Shows connectivity status.  "connectivity help" for more details

cras [ enable <flag> | disable <flag> | telephony <event> [arg] ]  
  Interact with CRAS(CrOS Audio Server) to set flags or trigger events.

  Subcommands:
    enable/disable <flag>:
      Enable or disable feature flag inside CRAS.
      Available flags:
      - wbs: Turn on this flag to make CRAS try to use wideband speech mode in
        Bluetooth HFP, if both the Bluetooth controller and peripheral supports
        this feature.
      - noise_cancellation: Turn on this flag to enable noise cancellation
        processing to input devices who support noise cancellation.

      Notice: The flag changes made through this command do not persist after
      reboot.

    telephony <event> [args...]:
      Trigger a telephony event pass to CRAS in Bluetooth HFP.

      Available events:
      - IncomingCall <PhoneNumber>: Make a call from external line.
          Ex: $ cras telephony IncomingCall 12345678
      - AnswerCall: Answer the call.
      - TerminateCall: Terminate the call.
      - SetDialNumber [DailNumber]: Set the last dialed number recorded. Clear
        the recorded number if no DailNumber is provided.
      - SetBatteryLevel <BatteryLevel>: Set and transfer the battery level to
        HF. The BatteryLevel should range from 0 to 5.
      - SetSignalStrength <SignalStrength>: Set and transfer the signal strength
        to HF. The SignalStrength should range from 0 to 5.
      - SetServiceAvailability <0/1>: Set if the service is available or not.
      - SetCallheld <CallHeldIndicator>: Set the callheld indicator.
        CallHeldIndicator:
          0 = No calls held
          1 = Call is placed on hold or active/held calls swapped
          (The AG has both an active AND a held call)
          2 = Call on hold, no active call
      - SetCallsetup <CallSetupIndicator>: Set the callsetup indicator.
        CallSetupIndicator:
          0 = No call setup in progress
          1 = Incoming call setup in progress
          2 = Outgoing call setup in dialing state
          3 = Outgoing call setup in alerting state
      - SetCall <CallIndicator>: Set the call indicator.
        CallIndicator:
          0 = No call (held or active)
          1 = Call is present (active or held)

diag
Usage: [list|routine]

display_debug
  A tool to assist with collecting logs when reproducing display related issues.

  Subcommands:
    trace_start Usage: display_debug trace_start
      Increase size and verbosity of logging through drm_trace.

    trace_stop Usage: display_debug trace_stop
      Reset size and verbosity of logging through drm_trace to defaults.

    trace_annotate Usage: display_debug trace_annotate <message>
      Append |message| to the drm_trace log.

    diagnose Usage: display_debug diagnose
      Give a sequence of steps to assist in debugging display issues.

dlc_install <dlc-id>
  Trigger a DLC installation against a **test** update server.

The **test** update server will serve signed DLC payloads which will only
successfully install if signed and verifiable. Otherwise, installations will
fail. The magical string 'autest' is passed into update_engine during this
installation, which will make requests against QA Omaha server and not the
production Omaha server.

WARNING: This may install an untested version of the DLC which was never
intended for end users!

Usage: dmesg [options]
        -d, --show_delta
                Display the timestamp and the time delta
            spent between messages
        -H, --human
                Enable human-readable output.
        -k, --kernel
                Print kernel messages.
        -L, --color
                Colorize the output.
        -p, --force-prefix
                Add facility, level or timestamp
            information to each line of a multi-line message.
        -r, --raw
                Print the raw message buffer.
        -T, --ctime
                Print human-readable timestamps.
        -t, --notime
                Do not print kernel's timestamps.
        -u, --userspace
                Print userspace messages.
        -x, --decode
                Decode facility and level (priority) numbers
            to human-readable prefixes.
dump_emk   
  Show the EMK (Enterprise Machine Key).

enroll_status [--mode] [--domain] [--realm] [--user]  
  Displays device enrollment information.

evtest   
  Run evtest in safe mode.

help: unknown command 'exit'
ff_debug
Usage: ff_debug [<tag_expression>]|[--reset]|[--help]|[--list_valid_tags]|[--level <level>]

  ff_debug adds and removes debug tags for flimflam.
  Current debug settings are displayed if no parameters are provided

  <tag_expression> is defined in boolean notation using <debug_tag> separated
    by an operator [+-], where + and - imply adding and removing of the tag immediately
    following the operator. An expression beginning with either operators [+-]
    takes the existing tags and modifies them appropriately.  Otherwise, the existing tags
    are replaced by those specified in the command.

    <debug_tag> can be listed using the --list_valid_tags

    e.g.: ff_debug network+wifi
      Sets debug tags to network and wifi
    e.g.: ff_debug +network-service
      Adds network and removes service tags from the existing debug settings

  --list_valid_tags : Displays all valid tags

  --level: Displays or sets current debug level for logging
    All messages at, or above, the current log level are logged. Normal log
    levels range from 4 (LOG_FATAL) to 0 (LOG_INFO). In addition VERBOSE log
    levels are available starting at -1.

    e.g.: ff_debug --level 4
      Logs only FATAL messages.
    e.g.: ff_debug --level 0
      Logs INFO, WARNING, ERROR, ERROR_REPORT, and FATAL messages.
    e.g.: ff_debug --level -4
      Logs everything that "--level 0" does, plus SLOG(<tag>, <n>) messages,
      where <tag> has been enabled for logging, and <n> <= 4. (NOTE: you must
      negate SLOG levels for use with ff_debug. In this example, SLOG(<tag>, 4)
      maps to "--level -4".)

  --reset : Removes all tagging

  --help : Displays this output

free [options]  
  Display free/used memory info

gesture_prop [ devices | list <device ID> | get <device ID> <property name> | set <device ID> <property name> <value> ]  
  Read and change gesture properties for attached input devices. The "Enable
  gesture properties D-Bus service" flag must be enabled for this command to
  work.

  For more details, see:
  https://chromium.googlesource.com/chromiumos/platform/gestures/+/HEAD/docs/gesture_properties.md

  Subcommands:
    devices, devs:
      List the devices managed by the gestures library, with their numeric IDs.

    list <device ID>:
      List the properties for the device with the given ID.

    get <device ID> <property name>:
      Get the current value of a property. Property names containing spaces
      should be quoted. For example:
      $ gesture_prop get 12 "Mouse CPI"

    set <device ID> <property name> <value>:
      Set the value of a property. Values use the same syntax as dbus-send
      (https://dbus.freedesktop.org/doc/dbus-send.1.html#description), and
      should either be arrays or strings. For example:
      $ gesture_prop set 12 "Mouse CPI" array:double:500
      $ gesture_prop set 12 "Log Path" string:/tmp/foo.txt

help [command]  
  Display general help, or details for a specific command.

help_advanced   
  Display the help for more advanced commands, mainly used for debugging.

Usage: hibernate [options]

Options:
    -n, --no-delay      skip the delay before initiating the hibernation.
ipaddrs [-4] [-6]  
  Display IP addresses.
  -4: Only show IPv4 addresses.
  -6: Only show IPv6 addresses.

meminfo   
  Display detailed memory statistics

memory_test   
  Performs extensive memory testing on the available free memory.

modem <command> [args...]  
  Interact with the 3G modem. Run "modem help" for detailed help.

network_diag
This command is disabled pending removal.
Please see https://issuetracker.google.com/267537630

p2p_update [enable|disable] [--num-connections] [--show-peers]  
  Enables or disables the peer-to-peer (P2P) sharing of updates over the local
  network. This will both attempt to get updates from other peers in the
  network and share the downloaded updates with them. Run this command without
  arguments to see the current state.  Additional switches will display number
  of connections and P2P peers.

Usage: packet_capture [options]
        --device        <device>
        --max-size      <max size in MiB>
        --frequency     <frequency>
        --ht-location   <above|below>
        --vht-width     <80|160>
        --monitor-connection-on         <monitored_device>

Start packet capture.  Start a device-based capture on <device>,
  or do an over-the-air capture on <frequency> with an optionally
  provided HT channel location or VHT channel width.  An over-the-air
  capture can also be initiated using the channel parameters of a
  currently connected <monitored_device>.  Note that over-the-air
  captures are not available with all 802.11 devices. Set <max_size>
  to stop the packet capture if the output .pcap file size exceedes this
  limit. Only device-based capture options (--device and --max-size) are
  available in verified mode. Switch to developer mode to use other
  options.
ping [-4] [-6] [-c count] [-i interval] [-n] [-s packetsize] [-W waittime] <destination>  
  Send ICMP ECHO_REQUEST packets to a network host.  If <destination> is "gw"
  then the next hop gateway for the default route is used.
  Default is to use IPv4 [-4] rather than IPv6 [-6] addresses.

printscan_debug
  A tool to assist with collecting logs when reproducing printing and scanning related issues.

  Subcommands:
    start Usage: printscan_debug start [all|scanning|printing]
      Start collecting printscan debug logs.

    stop Usage: printscan_debug stop
      Stop collecting printscan debug logs.

rlz < status | enable | disable >  
  Enable or disable RLZ.  See this site for details:
  http://dev.chromium.org/developers/design-documents/extensions/proposed-changes/apis-under-development/rlz-api

rollback
  
  Attempt to rollback to the previous update cached on your system. Only
  available on non-stable channels and non-enterprise enrolled devices.

  Please note that this will powerwash your device.

route [-4] [-6] [--all]  
  Display the routing tables.
  -4: Only show IPv4 routes.
  -6: Only show IPv6 routes.
  --all: Show all routing tables.

set_apn [-c] [-n <network-id>] [-u <username>] [-p <password>] <apn>  
  Set the APN to use when connecting to the network specified by <network-id>.
  If <network-id> is not specified, use the network-id of the currently
  registered network.

  The -c option clears the APN to be used, so that the default APN will be used
  instead.

set_arpgw <true | false>  
  Turn on extra network state checking to make sure the default gateway
  is reachable.

set_cellular_ppp [-c] [-u <username>] [-p <password>]  
  Set the PPP username and/or password for an existing cellular connection.
  If neither -u nor -p is provided, show the existing PPP username for
  the cellular connection.

  The -c option clears any existing PPP username and PPP password for an
  existing cellular connection.

set_time <time string>
  Sets the system time if the the system has been unable to get it from the
  network. The <time string> uses the format of the GNU coreutils date command.

set_wake_on_lan <true | false>  
  Enable or disable Wake on LAN for Ethernet devices.  This command takes
  effect after re-connecting to Ethernet and is not persistent across system
  restarts.

storage_test_1   
  Performs a short offline SMART test.

storage_test_2   
  Performs an extensive readability test.

swap [ enable <size (MB)> | disable | start | stop | status ]  
  Change kernel memory manager parameters
  (FOR EXPERIMENTS ONLY --- USE AT OWN RISK)

  "swap status" (also "swap" with no arguments) shows the values of various
  memory manager parameters and related statistics.

  The enable/disable options enable or disable compressed swap (zram)
  persistently across reboots, and take effect at the next boot.  The enable
  option takes the size of the swap area (in megabytes before compression).
  If the size is omitted, the factory default is chosen.

  The start/stop options turn swap on/off immediately, but leave the settings
  alone, so that the original behavior is restored at the next boot.

  WARNING: if swap is in use, turning it off can cause the system to
  temporarily hang while the kernel frees up memory.  This can take
  a long time to finish.

sync   
  Run the sync command

syslog <message>  
  Logs a message to syslog (the system log daemon).

time_info   
  Returns the current synchronization state for the time service.

top   
  Run top.

tracepath [-4] [-6] [-n] <destination>[/port]  
  Trace the path/route to a network host.
  Default is to trace IPv4 [-4] rather than IPv6 [-6] targets.

u2f_flags <u2f | g2f>[, user_keys, verbose]  

  ### IMPORTANT: The U2F feature is experimental and not suitable for
  ### general production use in its current form. The current
  ### implementation is still in flux and some features (including
  ### security-relevant ones) are still missing. You are welcome to
  ### play with this, but use at your own risk. You have been warned.

  Set flags to override the second-factor authentication daemon configuration.
  u2f: Always enable the standard U2F mode even if not set in device policy.
  g2f: Always enable the U2F mode plus some additional extensions.
  user_keys: Enable user-specific keys.
  verbose: Increase the daemon logging verbosity in /var/log/messages.

uname [-a|-s|-n|-r|-v|-m|-p|-i|-o]  
  Display system info

upload_crashes   
  Uploads available crash reports to the crash server.

upload_devcoredumps [enable|disable]  
  Enable or disable the upload of devcoredump reports.

uptime   
  Display uptime/load info

verify_ro
  Verify AP and EC RO firmware on a ChromeOS device connected over SuzyQ
  cable, if supported by the device.

USAGE: vmc
   [ start [--enable-gpu] [--enable-dgpu-passthrough] [--enable-vulkan] [--enable-big-gl] [--enable-virtgpu-native-context] [--enable-audio-capture] [--untrusted] [--extra-disk PATH] [--kernel PATH] [--initrd PATH] [--writable-rootfs] [--kernel-param PARAM] [--bios PATH] [--timeout PARAM] [--oem-string STRING] <name> |
     stop <name> |
     launch <name> |
     create [-p] [--size SIZE] <name> [<source media> [<removable storage name>]] [-- additional parameters] |
     create-extra-disk --size SIZE [--removable-media] <host disk path> |
     adjust <name> <operation> [additional parameters] |
     destroy [-y] <name> |
     disk-op-status <command UUID> |
     export [-d] [-f] <vm name> <file name> [<removable storage name>] |
     import [-p] <vm name> <file name> [<removable storage name>] |
     resize <vm name> <size> |
     list |
     logs <vm name> |
     share <vm name> <path> |
     unshare <vm name> <path> |
     container <vm name> <container name> [ (<image server> <image alias>) | (<rootfs path> <metadata path>)] [--privileged <true/false>] [--timeout PARAM] |
     update-container-devices <vm_name> <container_name> (<vm device>:<enable/disable>)... |
     usb-attach <vm name> <bus>:<device> [<container name>] |
     usb-detach <vm name> <port> |
     usb-list <vm name> |
     pvm.send-problem-report [-n <vm name>] [-e <reporter's email>] <description of the problem> |
     --help | -h ]

vmstat [-a|-d|-f|-m|-n|-s|-w] [delay [count]]  
  Report virtual memory statistics

vsh <vm_name> [<container_name>]  
  Connect to a shell inside the VM <vm_name>, or to a shell inside the container
  <container_name> within the VM <vm_name>.

wifi_fw_dump   
  Manually trigger a WiFi firmware dump. This command will currently only
  complete successfully if the intel-wifi-fw-dumper package is present on the
  device with USE=iwlwifi_dump enabled.

wifi_power_save < status | enable | disable >  
  Enable or disable WiFi power save mode.  This command is not persistent across
  system restarts.

```
