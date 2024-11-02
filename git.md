## git

### misc
```
git checkout -b other_branch origin/other_branch
git ls-files
git grep <term>
# addした変更だけを見たい時
git diff --staged
```

### fixup & autosquash
```
git config rebase.autoSquash true
git rebase --root -i
```

### 各コミットにまとめてテストを走らせる
```
git rebase HEAD~~ --exec "git show --oneline -s HEAD | grep SKIP_TEST: || cargo test"

git rebase --root --exec "git show --oneline -s HEAD | grep SKIP_TEST: || cargo test"
```

### patch関連
ユーザー名を変更する必要があるなら変更してコミットを修正
```
git config --global user.name "hikalium"
git config --global user.email "hikalium@hikalium.com"

git config --global --unset user.name
git config --global --unset user.email

git config --local user.name "hikalium"
git config --local user.email "hikalium@hikalium.com"

git commit --amend --reset-author
```

コミットをまとめる(nはまとめる個数)
```
git rebase -i HEAD~n
```

### email
- http://d.hatena.ne.jp/fixme/20110212/1297564806

### https password
https://qiita.com/usamik26/items/c655abcaeee02ea59695

```
git config --global credential.helper 'cache --timeout=3600' # 1h
git config --global credential.helper 'cache --timeout=86400' # 1d
```
