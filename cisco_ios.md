## Cisco IOS

- Cisco 800M Series (4ポート) で検証

`interface gigabitEthernet 0/4`を外向きのinterfaceに使っている。

`interface range GigabitEthernet 0/0-3` とかすれば、まとめて設定ができる

### よくある`no ip domain-lookup`ではない方法でコマンド入力をDNSにかけようとするのを止める
`ip domain-lookup`を`no`にすると、すべての解決が止まってしまってこまる。
私の場合、DDNSのアクセス先ドメインを解決できなくて困った。
`HTTPDNSUPD: Sending request... status='Host name resolution failed', tid=0`と出てしまう。
pingとかも解決してくれなくなる。これは困る。
`ip domain-lookup`を有効にすれば、正しく解決できるようになった。
- Solution 2 of http://etherealmind.com/cisco-ios-translating-domain-server/
- http://blog.livedoor.jp/tokyo_z/archives/51247251.html

つまりは、全部のlineに対して`transport preferred none`とすればよい。
```
ip domain-lookup
line vty 0 165
transport preferred none
```

### インターフェイスのIPとサブネットマスクを設定
```
ip address 192.168.1.1 255.255.255.0
```

### もしくは、dhcpで取得するよう設定する
```
ip address dhcp
```

### 各種確認
```
show ip dhcp pool
show interface
show ip interface
show vlan-switch
```

### ここが一番参考になりそう。試してみる。
- https://www.cisco.com/c/ja_jp/support/docs/ip/ip-addressing-services/dhcp.html

```
z01(config)#ip dhcp excluded-address 10.10.10.1
z01# show interfaces Vlan1
z01(config)#interface Vlan1
z01(config-if)#
```
### エラーでこけた
```
ip nat inside source list 1 interface gigabitEthernet0/4 overload
```
しようとしたら下記のエラーがでた
```
Dynamic mapping in use, cannot change
```
調べたところ、一度natをクリアする必要があるっぽい。
Vlan1で
```
no ip nat inside
```
その後、
```
clear ip nat translation *
```
その後、再度Vlan1で
```
ip nat inside source list 1 interface gigabitEthernet0/4 overload
```
したら通った。

### telnet禁止、内部ipのみssh可にしたい

- vtyの個数はcontext sensitive help (?キー) をみるのがよい

```
line vty 0 165
transport input ssh
access-class 1 in
```

- aclは適宜設定すること
- これでちゃんと防げた

### DNSポートも塞ぎたい
```
no ip dns server
```
### 外に出れなくて悩んだ結果
外に出れなかったのはファイアウォールのせいだった。
注意しよう。

### dhcpでiPXEブートしたい！
- https://raphine.wordpress.com/2016/12/14/netboot/

該当するDHCPプール(`ip dhcp pool ccp-pool`など)で以下を設定。
```
bootfile http://<server ip>:<port>/ipxe.conf
```

`ipxe.conf` には、ipxeターミナルで入力するのと同等の記述をしておけばよい。
ただし、最初の行に `#!ipxe` と書いておかないと実行されない。

- `ipxe.conf` sample
```
#!ipxe
dhcp
kernel http://<server ip>:<port>/memdisk
initrd http://<server ip>:<port>/disk.img.gz
boot
```

### dhcpで降ってくるDNSアドレスを変更したい！
該当するDHCPプールで
```
dns-server 8.8.8.8
```

## static nat
- 外部から内部へのNAT(外部がGigabitEthernet 0/4, 外部の1234を内部192.168.0.5:5678に転送)
```
ip nat inside source static tcp 192.168.0.5 5678 interface GigabitEthernet 0/4 1234
```

### 設定の保存
```
write memory
```
