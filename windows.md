# windows

https://learn.microsoft.com/ja-jp/windows-server/administration/openssh/openssh_install_firstuse?tabs=powershell&pivots=windows-11

```
curl.exe https://github.com/hikalium.keys > %USERPROFILE%\.ssh\authorized_keys
type %USERPROFILE%\.ssh\authorized_keys

type C:\ProgramData\ssh\sshd_config
notepad.exe C:\ProgramData\ssh\sshd_config
```

https://www.wireguard.com/install/

```
"C:\Program Files\WireGuard\wireguard.exe" /installtunnelservice "C:\path\to\your-config.conf"
```

```
Set-NetConnectionProfile -InterfaceAlias "wgN" -NetworkCategory Private
```
