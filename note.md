## Side notes

*Just experimenting with enviroment variables this is just a dumping playground*

**Looking into the addresss just pinning it here**
```
cardano-cli query utxo --address $(cat payment.addr) --testnet-magic $MAGICID --mary-era
```

export TXNID1=2ff2c2a5321502749ae1bc5133158872ac5e2f9b4a527fcef7801cab7a082878
export LOVELACE1=20000000

export OUTADDR=addr_test1qqykkqr28fvylu3el7zht6vx6ynt9d3dw5u5l2yfk8lv60l3zg5yyt7lc4wuekkks0pefg468s8nhy2e4srz7lu2dssqt0hta6

export FEE=0
export MINLOVELACE=5000000
export OUTFILE=matx.raw


*This is just me copying and pasting code and changing it up a bit.*

cardano-cli transaction build-raw \
    --mary-era \
    --fee $FEE \
    --tx-in $TXNID1#0 \
    --tx-out $OUTADDR+$MINLOVELACE \
    --tx-out $(cat payment.addr)+$(expr $LOVELACE1 - $MINLOVELACE - $FEE) \
    --out-file matx.raw

cardano-cli transaction calculate-min-fee \
--tx-body-file matx.raw \
--tx-in-count 1 \
--tx-out-count 1 \
--witness-count 1 \
--testnet-magic $MAGICID \
--protocol-params-file protocol.json

export FEE=172893

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
## More side notes

```
cardano-cli transaction build-raw \
  --mary-era \
  --fee $FEE \
  --tx-in "$TXNID1"#0 \
  --tx-out=$OUTADDR+$(expr $LOVELACE1 - $FEE)+"$ASSET1" \
  --mint="$ASSET1" \
  --out-file $OUT_FILE
```

---
## Meta-data templates

```
make sure to keep the quotation marks when your changing the parameters

{
    "any number here":{
       "ticker": "any four letters that abrieveiate your token name",
       "name": "can be anything, make sure it matches you token name otherwise it will be confussing",
       "description": "describe your token here",
       "homepage": "can be anything here",
       "address": "Anyadress you may want to put here"
    }
  }
```