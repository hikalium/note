# tools

## git
```
git checkout -b <new_branch> <remote/branch>
git remote add <remote_name> <remote_url>
git fetch <remote_name>
git checkout -b <local_new_branch> <remote>/<remote_branch>
```

## screen with serial
```
sudo screen <device> <baudrate>
# kill: Ctrl-a k y
```

## GhostScript
### show all fonts available
```
echo "(*) {==} 256 string /Font resourceforall" | gs -
```
