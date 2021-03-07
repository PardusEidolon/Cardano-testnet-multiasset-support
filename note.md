## Side notes

*Just experimenting with enviroment variables*

cardano-cli query utxo --address $(cat payment.addr) --testnet-magic $MAGICID --mary-era

export TXNID1=0661bb4bf90b25388c6647ac46289e41bae40b8ea19d2044c742e715e4002c9b
export LOVELACE1=1481488

export TXNID2=28fdb70edc9817baadef5e0ace47bebf5c07a1082d1da733e8658f789b7b5dcc
export LOVELACE2=1444443

export TXNID3=8794b5f38416ed30c2b29e7aee0e80ccd95d6df45035e544313705724185416c
export LOVELACE3=1629628

export TXNID4=e7e08dd07e9ceebe0e99e065745a90f86931731100c9e976363e0379489924a5
export LOVELACE4=1444443

export ASSET1="500000 73ca2dcefe5652d52dc111cfa16c84b08d9222cbcc6290944da548f3.fruitcoin"
export ASSET2="10 5dcd455c11659e8e584d2f3dbb3c93a0a299434b1c3d61a27361b1a4.carrot"
export ASSET3="2 6b8d07d69639e9413dd637a1a815a7323c69c86abbafb66dbfdb1aa7"
export ASSET4="100 5dcd455c11659e8e584d2f3dbb3c93a0a299434b1c3d61a27361b1a4.eggplant"

export FEE=0
export MINLOVELACE=2000000


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
