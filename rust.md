# Rust related note

## Cleanup toolchain installations except stable

```
rustup toolchain list | grep -v stable | xargs -I {} -- rustup toolchain remove {}
```
