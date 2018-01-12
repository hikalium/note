## linux common

### ld
```
ldconfig -v
```

## usermod add user to group
```
usermod -aG group user
```

## ssh shared connection
以下の設定を`~/.ssh/config`に追加すれば、切断後もコネクションが維持される（接続のたびにパスワードを入れなくていいので便利）。
```
ControlMaster auto
ControlPath ~/.ssh/mux-%r@%h:%p
ControlPersist yes
```
### 接続を手動で切りたい
- https://unix.stackexchange.com/questions/24005/how-to-close-kill-ssh-controlmaster-connections-manually

でも、たまに接続を落としたくなる場合がある（グループの設定をいじって再ログインしたい時に、接続が維持されていると再ログインされないので反映されないので困る）。

その場合は以下のコマンドを実行。
```
ssh -O stop <hostname>
```

### マイク入力の確認
```
arecord -l
```
