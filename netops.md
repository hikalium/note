# network operations

## NEC IX

```
stty -F /dev/ttyUSB0 -crtscts
socat - /dev/ttyUSB0,b9600
```
