# network operations

## netplan
```
sudo netplan get
sudo vi /etc/netplan/*.yaml
sudo netplan generate
sudo netplan apply
dig misskey.hikalium.dev
```

```
# backend is networkd
$ sudo cat /etc/netplan/00-hikalium.yaml
network:
  version: 2
  ethernets:
    primary-interface:
      match:
        macaddress: XX:XX:XX:XX:XX:XX
      optional: true
      dhcp4: true
      addresses:
        - "A.B.C.D/24"
    wireless:
      match:
        name: "w*"
      optional: true
sudo chmod 600 /etc/netplan/*.yaml
sudo chown root:root /etc/netplan/*.yaml
sudo netplan try --timeout 10
# press Enter to accept the change
```

## NEC IX

```
stty -F /dev/ttyUSB0 -crtscts
socat - /dev/ttyUSB0,b9600
```
