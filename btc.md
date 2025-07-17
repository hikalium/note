# bitcoin-core

```
RPCUSER=`uuidgen -r` ; RPCPASS=`uuidgen -r` ; echo "RPCUSER=$RPCUSER ; RPCPASS=$RPCPASS ; " ; ./bin/bitcoind -datadir=/work/.bitcoin -rpcuser=${RPCUSER} -rpcpassword=${RPCPASS} 

ssh -L 8332:localhost:8332 ${HOST} -- sleep 1h

./bin/bitcoin-cli -rpcconnect=localhost -rpcport=8332 -rpcuser=${RPCUSER} -rpcpassword=${RPCPASS} --getinfo
```

```
# .bitcoin/wallets/hoge/wallet.dat

./bin/bitcoin-cli loadwallet hoge
./bin/bitcoin-cli getbalance

```
