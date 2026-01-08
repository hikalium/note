## git

### git log --format='...'

From `man git-log`:

| format_syntax | description |
|---|---|
| `%H` | commit hash |
| `%h` | abbreviated commit hash |
| `%T` | tree hash |
| `%t` | abbreviated tree hash |
| `%P` | parent hashes |
| `%p` | abbreviated parent hashes |
| `%an` | author name |
| `%aN` | author name (respecting .mailmap) |
| `%ae` | author email |
| `%aE` | author email (respecting .mailmap) |
| `%al` | author email local-part |
| `%aL` | author local-part (respecting .mailmap) |
| `%ad` | author date (format respects --date= option) |
| `%aD` | author date, RFC2822 style |
| `%ar` | author date, relative |
| `%at` | author date, UNIX timestamp |
| `%ai` | author date, ISO 8601-like format |
| `%aI` | author date, strict ISO 8601 format |
| `%as` | author date, short format (YYYY-MM-DD) |
| `%ah` | author date, human style |
| `%cn` | committer name |
| `%cN` | committer name (respecting .mailmap) |
| `%ce` | committer email |
| `%cE` | committer email (respecting .mailmap) |
| `%cl` | committer email local-part |
| `%cL` | committer local-part (respecting .mailmap) |
| `%cd` | committer date (format respects --date= option) |
| `%cD` | committer date, RFC2822 style |
| `%cr` | committer date, relative |
| `%ct` | committer date, UNIX timestamp |
| `%ci` | committer date, ISO 8601-like format |
| `%cI` | committer date, strict ISO 8601 format |
| `%cs` | committer date, short format (YYYY-MM-DD) |
| `%ch` | committer date, human style |
| `%d` | ref names, like the --decorate option of git-log(1) |
| `%D` | ref names without the " (", ")" wrapping |
| `%S` | ref name given on the command line by which the commit was reached |
| `%e` | encoding |
| `%s` | subject |
| `%f` | sanitized subject line, suitable for a filename |
| `%b` | body |
| `%B` | raw body (unwrapped subject and body) |
| `%N` | commit notes |
| `%GG` | raw verification message from GPG for a signed commit |
| `%G?` | show signature validity |
| `%GS` | show the name of the signer for a signed commit |
| `%GK` | show the key used to sign a signed commit |
| `%GF` | show the fingerprint of the key used to sign a signed commit |
| `%GP` | show the fingerprint of the primary key |
| `%GT` | show the trust level for the key used to sign a signed commit |
| `%gD` | reflog selector |
| `%gd` | shortened reflog selector |
| `%gn` | reflog identity name |
| `%gN` | reflog identity name (respecting .mailmap) |
| `%ge` | reflog identity email |
| `%gE` | reflog identity email (respecting .mailmap) |
| `%gs` | reflog subject |
| `%n` | newline |
| `%%` | a raw % |
| `%x00` | print a byte from a hex code |
| `%Cred` | switch color to red |
| `%Cgreen` | switch color to green |
| `%Cblue` | switch color to blue |
| `%Creset` | reset color |
| `%m` | left, right or boundary mark |

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
