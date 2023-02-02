# crosvm hack
```
RUSTFLAGS='-C target-feature=+crt-static' cargo build --target x86_64-unknown-linux-gnu --release
RUSTFLAGS='-C target-feature=+crt-static' cargo install --target x86_64-unknown-linux-gnu --path .
```
