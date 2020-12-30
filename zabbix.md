# zabbix

## agent installation

```
sudo apt install zabbix-agent

# edit/etc/zabbix/zabbix_agentd.conf
Server=10.10.10.110

sudo systemctl restart zabbix-agent.service
```

## check agent connectivity

```
# on zabbix server
zabbix_get -s <host> -k "system.uptime"
```

## Give permission to run a command with sudo from zabbix

```
sudo EDITOR=vim visudo

Defaults:zabbix    !requiretty
zabbix  ALL=NOPASSWD: /usr/bin/ipmctl
```
