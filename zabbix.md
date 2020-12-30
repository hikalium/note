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
