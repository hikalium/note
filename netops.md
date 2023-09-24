# network operations

## netplan
```
sudo netplan get
sudo vi /etc/netplan/*.yaml
sudo netplan generate
sudo netplan apply
dig misskey.hikalium.dev
```

## NEC IX

```
stty -F /dev/ttyUSB0 -crtscts
socat - /dev/ttyUSB0,b9600
```
