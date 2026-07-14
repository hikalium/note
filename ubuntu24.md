```
sudo sed -i.bak -E \
  '/^URIs:/ {
    N; /Suites:.*-security/! {
      s@http://(jp\.)?archive\.ubuntu\.com/ubuntu/?@http://ftp.udx.icscoe.jp/Linux/ubuntu/@;
      s@http://(jp\.)?ports\.ubuntu\.com/ubuntu-ports/?@http://ftp.udx.icscoe.jp/Linux/ubuntu-ports/@;
    }
  }' \
  /etc/apt/sources.list.d/ubuntu.sources
```

- [Ubuntuの日本国内向けapt mirror設定2026](https://zenn.dev/ciffelia/articles/c394962a8f188a)
