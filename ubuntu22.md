# Ubuntu 22.04 tips

```
$ cat /etc/lsb-release 
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=22.04
DISTRIB_CODENAME=jammy
DISTRIB_DESCRIPTION="Ubuntu 22.04.3 LTS"
```

```
$ sudo cat /etc/netplan/00-installer-config.yaml 
network:
  ethernets:
    enp1s0:
      dhcp4: false
      addresses: [${IP}/${PREFIX_LEN}, ${IP}/${PREFIX_LEN}]
      optional: true
      routes:
        - to: default
          via: ${IP}
      nameservers:
        addresses: [${IP}, 8.8.8.8, 4.4.4.4]
    enp2s0:
      dhcp4: true
      optional: true
  version: 2
```

See also: [netops.md](netops.md)

```
# update timezone
sudo dpkg-reconfigure tzdata
# check bottleneck of the boottime
sudo systemd-analyze blame
```
