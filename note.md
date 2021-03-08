## Side notes

*Just experimenting with enviroment variables*

**Looking into the addresss just pinning it here**
```
cardano-cli query utxo --address $(cat payment.addr) --testnet-magic $MAGICID --mary-era
```

export TXNID1=6e8460c6662d0d664e008dd3bbe2d2fe4f83464c0a0138f3560ee3c4f4b0ef0d
export LOVELACE1=50000000

export ASSET1="500000 73ca2dcefe5652d52dc111cfa16c84b08d9222cbcc6290944da548f3.fruitcoin"

export FEE=0
export MINLOVELACE=1444443


*This is just me copying and pasting code and changing it up a bit.*

cardano-cli transaction build-raw \
    --mary-era \
    --fee $FEE \
    --tx-in $TXNID1#0 \
    --tx-in $TXNID2#0 \
    --tx-in $TXNID3#0 \
    --tx-in $TXNID4#0 \
    --tx-out $(cat payment.addr)+$(expr $LOVELACE1 + $LOVELACE2 + $LOVELACE3 + $LOVELACE4 - $MINLOVELACE - $FEE) \
    --tx-out $(cat payment.addr)+$MINLOVELACE+"$ASSET1"+"$ASSET2"+"$ASSET3"+"$ASSET4" \
    --out-file matx.raw

cardano-cli transaction calculate-min-fee \
--tx-body-file matx.raw \
--tx-in-count 4 \
--tx-out-count 2 \
--witness-count 1 \
--testnet-magic $MAGICID \
--protocol-params-file protocol.json

export FEE=190317

cardano-cli transaction sign \
  --signing-key-file payment.skey \
  --testnet-magic $MAGICID \
  --tx-body-file matx.raw \
  --out-file matx.signed

cardano-cli transaction submit --tx-file matx.signed --testnet-magic $MAGICID

cardano-cli query utxo --address $(cat payment.addr) --testnet-magic $MAGICID --mary-era


---

cardano-cli query utxo --address $(cat payment.addr) --testnet-magic $MAGICID --mary-era

export TXNID1=f3a8b8d377211ab1687c4ba80cb6864df1823d47af3b3ef59575389b876ef354
export LOVELACE1=3809685

export FEE=0
export SENDADRESS=addr_test1qqykkqr28fvylu3el7zht6vx6ynt9d3dw5u5l2yfk8lv60l3zg5yyt7lc4wuekkks0pefg468s8nhy2e4srz7lu2dssqt0hta6

cardano-cli transaction build-raw \
    --mary-era \
    --fee $FEE \
    --tx-in $TXNID1#0 \
    --tx-out $SENDADRESS+$(expr $LOVELACE1 - $FEE) \
    --out-file matx.raw

cardano-cli transaction calculate-min-fee \
--tx-body-file matx.raw \
--tx-in-count 1 \
--tx-out-count 1 \
--witness-count 1 \
--testnet-magic $MAGICID \
--protocol-params-file protocol.json

export FEE=171265

cardano-cli transaction build-raw \
    --mary-era \
    --fee $FEE \
    --tx-in $TXNID1#0 \
    --tx-out $SENDADRESS+$(expr $LOVELACE1 - $FEE) \
    --out-file matx.raw

cardano-cli transaction sign \
  --signing-key-file payment.skey \
  --testnet-magic $MAGICID \
  --tx-body-file matx.raw \
  --out-file matx.signed

cardano-cli transaction submit --tx-file matx.signed --testnet-magic $MAGICID

cardano-cli query utxo --address $(cat payment.addr) --testnet-magic $MAGICID --mary-era

---