# bluetoothctl
```
sudo apt install -y bluez
sudo systemctl --no-pager status -l bluetooth
sudo systemctl restart bluetooth
sudo bluetoothctl --timeout 3 scan on && sudo bluetoothctl devices

```
