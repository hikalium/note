# Supermicro BMCの秘伝のタレ

## SSH Proxyした先のBMCにブラウザからうまくアクセスできないので原因調査
とりあえずSSH Proxy先の同一ネットワークにいるPCからfirefoxを起動してBMCにアクセスする
手元は[XQuartz](https://www.xquartz.org/)を入れておいて、X forwardingする。
