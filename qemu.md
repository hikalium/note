```
./configure --target-list=x86_64-softmmu
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

## commits
- https://github.com/qemu/qemu/commit/d5dbde4645fe56a1bcd678f85fa26c5548bcf552
- https://github.com/qemu/qemu/commit/5e9aa92eb1a5abbb6e0e3dafdf64ac728e11b6f2
