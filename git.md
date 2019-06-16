## git
```
git checkout -b other_branch origin/other_branch
git ls-files
git grep <term>
```

### patch関連
ユーザー名を変更する必要があるなら変更してコミットを修正
```
git config --global --unset user.name
git config --global --unset user.email
git config --local user.name "My Name"
git config --local user.email "me@example.com"
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
