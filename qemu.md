https://wiki.qemu.org/Contribute/SubmitAPatch
https://bugs.launchpad.net/qemu/

```
./configure --target-list=x86_64-softmmu
# for debugging: ./configure --target-list=x86_64-softmmu --enable-debug --enable-sanitizers
make
sudo make install
```

### setup git
```
git config diff.renames true
git config diff.algorithm patience
git config sendemail.cccmd 'scripts/get_maintainer.pl --nogit-fallback'
```

https://git-scm.com/docs/git-send-email#_use_gmail_as_the_smtp_server

### make patch
```
git format-patch -s HEAD~
```

### check patch
```
scripts/checkpatch.pl <patchfile>
```

### send!
- https://www.freedesktop.org/wiki/Software/PulseAudio/HowToUseGitSendEmail/
```
git send-email --to qemu-devel@nongnu.org <patchfile>
```

sample:
```
git send-email --smtp-debug=1 --to qemu-devel@nongnu.org 0001-x.patch
```

## commits

- https://github.com/qemu/qemu/search?q=hikaru&type=Commits
