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
