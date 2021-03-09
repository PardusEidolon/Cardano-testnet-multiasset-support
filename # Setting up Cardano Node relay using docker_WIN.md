# How to use Multi-Asset Tokens in the Cardano Node Testnet with docker on windows
---

## Setting up and running a cardano node on the mary testnet

1. First download and install docker: https://www.docker.com/products/docker-desktop
2. Create a cardano node folder on your main drive
    - *Inside create the subfolders "mary-data" & "mary-ipc" I just happen to have made a directory on my local C:\ drive for mac users just changes the appropriate paths*
3. Run the **cardano-node** below with the appropriate paths.
   - *you can name your image whatever you want to be honest*

```
docker run --detach --name=relay1 -v C:\docker\cardano-node\mary-ipc:/ipc -v C:\docker\cardano-node\mary-config:/config -e NETWORK=testnet inputoutput/cardano-node run
```

a. To run a shell in VS Code, setting the needed environment varibles this is optional you can just do this in the docker CLI.

```
docker exec -it -e CARDANO_NODE_SOCKET_PATH=/ipc/node.socket -e MAGICID=1097911063 relay1 sh
```

1. Now that we have a container running and all set up, export the cardano socket.
```
export CARDANO_NODE_SOCKET_PATH=/ipc/node.socket
```

5. Next run this line of code to query the tip of the node.
      - *To find the magic number, there should be a file located in your mar-ipc folder, alternativley you can find the magic number here https://hydra.iohk.io/build/5367762/download/1/index.html just select the testnet shellygenesis and look for "networkMagic"*

```
cardano-cli query tip --testnet-magic $MAGICID
```

Just checking the current slot ^

6. Cd into the config folder and generate your key pairs

```
cardano-cli address key-gen --verification-key-file payment.vkey --signing-key-file payment.skey
```

```
cardano-cli address build --payment-verification-key-file payment.vkey --out-file payment.addr --testnet-magic $MAGICID
```

7. Just bear in mind you have to wait for the node to sync up with the rest of the block otherwise youll run into errors typically have to wait 30 minutes to an hour

```
cardano-cli query utxo --address $(cat payment.addr) --testnet-magic $MAGICID --mary-era
```

8. Export the porotocol parameters

```
cardano-cli  query protocol-parameters --testnet-magic $MAGICID --out-file protocol.json --mary-era
```

9. Start the minting process

```
mkdir policy

cardano-cli address key-gen --verification-key-file policy/policy.vkey --signing-key-file policy/policy.skey
```

```
touch policy/policy.script && echo "{" > policy/policy.script

echo "  \"keyHash\": \"$(cardano-cli address key-hash --payment-verification-key-file policy/policy.vkey)\"," >> policy/policy.script

echo "  \"type\": \"sig\"" >> policy/policy.script

echo "}" >> policy/policy.script

cat policy/policy.script 

```

```
cardano-cli transaction policyid --script-file ./policy/policy.script 
```
policy script output: ca9d41124ec5508fb382996d480781c80f6f24cf0a9e2ca97630af69

## Build the transaction

1. build a raw transaction

```
cardano-cli transaction build-raw \
    --mary-era --fee 0 \
    --tx-in 606a2e022ee27dd7fb13a5f87059335875afc3ea094ef27883573c304f705062#0 \
    --tx-out addr_test1qqykkqr28fvylu3el7zht6vx6ynt9d3dw5u5l2yfk8lv60l3zg5yyt7lc4wuekkks0pefg468s8nhy2e4srz7lu2dssqt0hta6+10000000+"100 5dcd455c11659e8e584d2f3dbb3c93a0a299434b1c3d61a27361b1a4.eggplant" \
    --mint="100 5dcd455c11659e8e584d2f3dbb3c93a0a299434b1c3d61a27361b1a4.eggplant" \
    --out-file matx.raw
```

2. calculate fee

```
cardano-cli transaction calculate-min-fee \
--tx-body-file matx.raw \
--tx-in-count 1 \
--tx-out-count 1 \
--witness-count 2 \
--testnet-magic $MAGICID \
--protocol-params-file protocol.json
```

3. Build the transaction again

```
cardano-cli transaction build-raw \
    --mary-era --fee 181693 \
    --tx-in 606a2e022ee27dd7fb13a5f87059335875afc3ea094ef27883573c304f705062#0 \
    --tx-out addr_test1qqykkqr28fvylu3el7zht6vx6ynt9d3dw5u5l2yfk8lv60l3zg5yyt7lc4wuekkks0pefg468s8nhy2e4srz7lu2dssqt0hta6+9818307+"100 5dcd455c11659e8e584d2f3dbb3c93a0a299434b1c3d61a27361b1a4.eggplant" \
    --mint="100 5dcd455c11659e8e584d2f3dbb3c93a0a299434b1c3d61a27361b1a4.eggplant" \
    --out-file matx.raw
```

4. Sign the transaction

```
cardano-cli transaction sign \
	     --signing-key-file payment.skey \
	     --signing-key-file policy/policy.skey \
	     --script-file policy/policy.script \
	     --testnet-magic $MAGICID \
	     --tx-body-file matx.raw \
         --out-file matx.signed

```

## Submit Transaction

```
cardano-cli transaction submit --tx-file  matx.signed --testnet-magic $MAGICID
```

1. now check the transaction is completed

```
cardano-cli query utxo --address addr_test1qqykkqr28fvylu3el7zht6vx6ynt9d3dw5u5l2yfk8lv60l3zg5yyt7lc4wuekkks0pefg468s8nhy2e4srz7lu2dssqt0hta6 --testnet-magic $MAGICID --mary-era
```

---
## Creating a carrot coin with description and two outputs

These instructions create a transaction with two transaction outputs. In practise the
same can be achieve by using a single transaction output with three assets (I think).

```
cardano-cli transaction build-raw \
    --mary-era --fee 0 \
    --tx-in 5a8941a265e79224141aaa833f229fbfb50f7ba59ddde379a697a81d2cb64e03#0 \
    --tx-out addr_test1qqykkqr28fvylu3el7zht6vx6ynt9d3dw5u5l2yfk8lv60l3zg5yyt7lc4wuekkks0pefg468s8nhy2e4srz7lu2dssqt0hta6+1444443+"10 5dcd455c11659e8e584d2f3dbb3c93a0a299434b1c3d61a27361b1a4.carrot" \
    --tx-out $(cat payment.addr)+8555557 \
    --mint="10 5dcd455c11659e8e584d2f3dbb3c93a0a299434b1c3d61a27361b1a4.carrot" \
    --out-file matx-carrot.raw
```

2. calculate fee

```
cardano-cli transaction calculate-min-fee \
--tx-body-file matx-carrot.raw \
--tx-in-count 1 \
--tx-out-count 1 \
--witness-count 2 \
--testnet-magic $MAGICID \
--protocol-params-file protocol.json
```

Output: 182881 Lovelace

3. Build the transaction again

```
cardano-cli transaction build-raw \
    --mary-era --fee 182881 \
    --tx-in 5a8941a265e79224141aaa833f229fbfb50f7ba59ddde379a697a81d2cb64e03#0 \
    --tx-out addr_test1qqykkqr28fvylu3el7zht6vx6ynt9d3dw5u5l2yfk8lv60l3zg5yyt7lc4wuekkks0pefg468s8nhy2e4srz7lu2dssqt0hta6+1444443+"10 5dcd455c11659e8e584d2f3dbb3c93a0a299434b1c3d61a27361b1a4.carrot" \
    --tx-out $(cat payment.addr)+8372676 \
    --mint="10 5dcd455c11659e8e584d2f3dbb3c93a0a299434b1c3d61a27361b1a4.carrot" \
    --out-file matx-carrot.raw
```

4. Sign the transaction

```
cardano-cli transaction sign \
	     --signing-key-file payment.skey \
	     --signing-key-file policy/policy.skey \
	     --script-file policy/policy.script \
	     --testnet-magic $MAGICID \
	     --tx-body-file matx-carrot.raw \
         --out-file matx-carrot.signed

```

## Submit Transaction

```
cardano-cli transaction submit --tx-file  matx-carrot.signed --testnet-magic $MAGICID
```

1. now check the transaction is completed

```
cardano-cli query utxo \
  --address addr_test1qqykkqr28fvylu3el7zht6vx6ynt9d3dw5u5l2yfk8lv60l3zg5yyt7lc4wuekkks0pefg468s8nhy2e4srz7lu2dssqt0hta6 \
  --address $(cat payment.addr) \
 --testnet-magic $MAGICID --mary-era
```

5dcd455c11659e8e584d2f3dbb3c93a0a299434b1c3d61a27361b1a4.carrot

Carrots are great for the eyesight.

---

## Building a transaction with meta-data

1. Setting up enviroment varibles

```
export TXNID1=eebd0033bcd1cb35dee40441a82bb71709c76e47ae3d2fe98f9c94e13d2086f5
export LOVELACE1=20000000

export ASSET1="10 ca9d41124ec5508fb382996d480781c80f6f24cf0a9e2ca97630af69.alphaCoin"
export METADATA_FILE=./metadata.json

export OUTADDR=addr_test1qqykkqr28fvylu3el7zht6vx6ynt9d3dw5u5l2yfk8lv60l3zg5yyt7lc4wuekkks0pefg468s8nhy2e4srz7lu2dssqt0hta6

export OUT_FILE=matx.raw
export SIGNED_FILE=matx.signed

export FEE=0
export MINLOVELACE=0
```

1. Build the transaction

```
cardano-cli transaction build-raw \
  --mary-era \
  --fee $FEE \
  --tx-in "$TXNID1"#0 \
  --tx-out=$OUTADDR+$(expr $LOVELACE1 - $FEE)+"$ASSET1" \
  --mint="$ASSET1" \
  --json-metadata-no-schema \
  --metadata-json-file $METADATA_FILE \
  --out-file $OUT_FILE
```

2. Calculate min fee

```
cardano-cli transaction calculate-min-fee \
--tx-body-file $OUT_FILE \
--tx-in-count 1 \
--tx-out-count 1 \
--witness-count 1 \
--testnet-magic $MAGICID \
--protocol-params-file protocol.json
```

3. build the transaction again with the new fee

```
export FEE=183849
``` 

4. Sign the transaction then rebuild the transaction again

```
cardano-cli transaction sign \
	     --signing-key-file payment.skey \
	     --signing-key-file policy/policy.skey \
	     --script-file policy/policy.script \
	     --testnet-magic $MAGICID \
	     --tx-body-file $OUT_FILE \
         --out-file $SIGNED_FILE

```

5. Submit the transaction

```
cardano-cli transaction submit --tx-file  $SIGNED_FILE --testnet-magic $MAGICID
```

6. check out the transaction :D
```
cardano-cli query utxo --address $(cat payment.addr) --testnet-magic $MAGICID --mary-era
```
---
## Burning Tokens

In order to burn tokens you have to make sure you have the same policyid, skey and vkey from which the minted tokens came from otherwise you will not be able to complete this proccess.

1. lets not forget to set up out enviroment varibles

```
export TXNID1=73df5eaa541e9a3dac8e4f2e92f917f666732db68add15ca9d6ec81746236f65
export LOVELACE1=2000000

export ASSET1="-10 ca9d41124ec5508fb382996d480781c80f6f24cf0a9e2ca97630af69.alphaCoin"

export ADDR=addr_test1vzp8fywldcewta76h5lz0g85c8de9djqgn0pa5m8z2tvjhq4d6ezc
export OUT_ADDR=addr_test1qqykkqr28fvylu3el7zht6vx6ynt9d3dw5u5l2yfk8lv60l3zg5yyt7lc4wuekkks0pefg468s8nhy2e4srz7lu2dssqt0hta6

export OUT_FILE=matx.raw
export SIGNED_FILE=matx.signed

export FEE=0
export MINLOVELACE=0
```

2. first we build our transaction

```
cardano-cli transaction build-raw \
        	--mary-era \
        	--fee $FEE \
        	--tx-in "$TXNID1"#0 \
            --tx-out=$OUT_ADDR+$(expr $LOVELACE1 - $FEE) \
         	--mint="$ASSET1" \
       	    --out-file $OUT_FILE
```

3. Generate a fee then build the transaction again after exporting the new fee varible

```
cardano-cli transaction calculate-min-fee \
--tx-body-file $OUT_FILE \
--tx-in-count 1 \
--tx-out-count 1 \
--witness-count 1 \
--testnet-magic $MAGICID \
--protocol-params-file protocol.json
```

export FEE=173377

4. Sign Transaction

```
cardano-cli transaction sign \
	     --signing-key-file payment.skey \
	     --signing-key-file policy/policy.skey \
	     --script-file policy/policy.script \
	     --testnet-magic $MAGICID \
	     --tx-body-file $OUT_FILE \
         --out-file $SIGNED_FILE

```

5. Submit the transaction

```
cardano-cli transaction submit --tx-file  $SIGNED_FILE --testnet-magic $MAGICID
```

6. check out the transaction :D
```
cardano-cli query utxo --address $(cat payment.addr) --testnet-magic $MAGICID --mary-era
```