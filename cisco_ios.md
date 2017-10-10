## Cisco IOS

- Cisco 800M Series (4ポート) で検証

interface gigabitEthernet 0/4
で外向きインターフェイスの設定に入れる。

# このインターフェイスのIPとサブネットマスクを設定
ip address 192.168.1.1 255.255.255.0

# もしくは、dhcpで取得するよう設定する
ip address dhcp

show ip dhcp pool
show interface
show ip interface


# ここが一番参考になりそう。試してみる。
https://www.cisco.com/c/ja_jp/support/docs/ip/ip-addressing-services/dhcp.html

z01(config)#ip dhcp excluded-address 10.10.10.1
z01# show interfaces Vlan1
z01(config)#interface Vlan1
z01(config-if)#



ip nat inside source list 1 interface gigabitEthernet0/4 overload
しようとしたら下記のエラーがでた
Dynamic mapping in use, cannot change
調べたところ、一度natをクリアする必要があるっぽい。
Vlan1で
no ip nat inside

その後、
z01#clear ip nat translation *

これでもういっかいVlan1で
ip nat inside source list 1 interface gigabitEthernet0/4 overload
したら通った。

show vlan-switch




# telnet禁止、内部ipのみssh可
# vtyの個数はcontext sensitive help (?キー) をみるのがよい
line vty 0 165
transport input ssh
access-class 1 in

これでちゃんと防げた

no ip dns server
