# NTP

```
sudo apt install chronyd
```

```
echo 'server ntp.nict.jp iburst' | sudo tee /etc/chrony/sources.d/ntp-nict-jp.sources
```

```
chronyc sources
chronyc sources -v
chronyc -N sources
chronyc -N -c sources
```

```
$ chronyc -N sources
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================
^- ntp.ubuntu.com                2   6    17    17  +8662us[+8362us] +/-  106ms
^- ntp.ubuntu.com                2   6    17    16  +4088us[+4088us] +/-  110ms
^- ntp.ubuntu.com                2   6    17    17  +3713us[+3414us] +/-  110ms
^- ntp.ubuntu.com                2   6    17    15  +3780us[+3780us] +/-  110ms
^* 0.ubuntu.pool.ntp.org         2   6    17    16   -324us[ -624us] +/- 4350us
^+ 1.ubuntu.pool.ntp.org         1   6    17    17    +42us[ -257us] +/- 3548us
^- 2.ubuntu.pool.ntp.org         3   6    17    16  -1574us[-1574us] +/-  116ms
^+ 2.ubuntu.pool.ntp.org         3   6    17    17   +334us[  +35us] +/- 5180us
^- ntp.nict.jp
```
