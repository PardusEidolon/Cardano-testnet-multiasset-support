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
      - *To find the magic number, there should be a file located in your mar-ipc folder, alternativley you can find the magic number here https://hydra.iohk.io/build/5367762/download/1/index.html just select the testnet shellygenesis and look for "networkMagic I just put the number into a varible to save time"*

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

7. Bear in mind if your getting an error here you have to wait for the node to sync up with the rest of the block otherwise youll keep running into errors. You typically have to wait 30 minutes to an hour

```
cardano-cli query utxo --address $(cat payment.addr) --testnet-magic $MAGICID --mary-era
```

8. Export the porotocol parameters

these are the network parameters need for the network to validate

```
cardano-cli  query protocol-parameters --testnet-magic $MAGICID --out-file protocol.json --mary-era
```

1. Start the minting process

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
policy script output: 67786695a6a87d36c1b6a8fd37275e1387ba3209c046bfbc742ae5f4

## Setting up our enviroment varibles

run the command cardano-cli query utxo --address $(cat payment.addr) --testnet-magic $MAGICID --mary-era to match the values below

```
export TXNID1="Name of the output"
export LOVELACE1= "how much love lace there is in the transaction out put"

export ASSET1="how much you want to mint plus the policy id plus the name"

export OUT_ADDR="which address you want to send too"

export OUT_FILE=matx.raw
export SIGNED_FILE=matx.signed

export FEE=0 "leave this at zero"
export MINLOVELACE=0 "minimum ammount of ada you want to send leave blank if you want to send the lot"
```

## Build the transaction

1. build a raw transaction

```
cardano-cli transaction build-raw \
    --mary-era \
    --fee $FEE
    --tx-in "$TXNID1"#0 \
    --tx-out $OUTADDR+$(expr $lOVElACE - $FEE)+"$ASSET1" \
    --mint="$ASSET1" \
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

export FEE="whatever the fee was calculated too"

3. Build the transaction again

```
cardano-cli transaction build-raw \
    --mary-era \
    --fee $FEE
    --tx-in "$TXNID1"#0 \
    --tx-out $OUTADDR+$(expr $lOVElACE - $FEE)+"$ASSET1" \
    --mint="$ASSET1" \
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
cardano-cli query utxo --address $(cat payment.addr) --testnet-magic $MAGICID --mary-era
```

---

## Building a transaction with meta-data

1. Make a json file in your current directory.

```
touch metadata.json
```

2. open up the file in your text editor and add the following template as a base just fil in the place holders

```
{
    "any number here":{
       "ticker": "any four letters that abrieveiate your token name",
       "name": "can be anything, make sure it matches you token name otherwise it will be confussing",
       "description": "describe your token here"
    }
  }
```


3. Setting up enviroment varibles

```
export TXNID1=9fb05135cdf95d2930d6119d45ea98df140d322114f8065ec7b54a920d208cd3
export LOVELACE1=14827107

export ASSET1="10 67786695a6a87d36c1b6a8fd37275e1387ba3209c046bfbc742ae5f4.lemonCoin"
export METADATA=./metadata.json

export OUTADDR=addr_test1qqykkqr28fvylu3el7zht6vx6ynt9d3dw5u5l2yfk8lv60l3zg5yyt7lc4wuekkks0pefg468s8nhy2e4srz7lu2dssqt0hta6

export OUT_FILE=matx.raw
export SIGNED_FILE=matx.signed

export FEE=0
export MINLOVELACE=2000000
```

1. Build the transaction

```
cardano-cli transaction build-raw \
  --mary-era \
  --fee $FEE \
  --tx-in "$TXNID1"#1 \
  --tx-out=$OUTADDR+$MINLOVELACE+"$ASSET1" \
  --tx-out=$(cat payment.addr)+$(expr $LOVELACE1 - $MINLOVELACE - $FEE ) \
  --mint="$ASSET1" \
  --json-metadata-no-schema \
  --metadata-json-file $METADATA \
  --out-file $OUT_FILE
```

2. Calculate min fee

```
cardano-cli transaction calculate-min-fee \
--tx-body-file $OUT_FILE \
--tx-in-count 1 \
--tx-out-count 2 \
--witness-count 1 \
--testnet-magic $MAGICID \
--protocol-params-file protocol.json
```

3. build the transaction again with the new fee

```
export FEE=184773
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

*this is a work in progess on the documentation at the moment below*

In order to burn tokens you have to make sure you have the same policyid, skey and vkey from which the minted tokens came from otherwise you will not be able to complete this proccess.

1. lets not forget to set up out enviroment varibles

```
export TXNID1=7daa9cb293a261db00af329bff094aced1db0a76c50e4c76adb11682b3389bda
export LOVELACE1=2000000

export TXNID2=b86fa1d95f5086d3d7efee33aa468fa235022d5747e7a976306d7ec9c97987f7
export LOVELACE2=12642334

export ASSET1="-10 67786695a6a87d36c1b6a8fd37275e1387ba3209c046bfbc742ae5f4.lemonCoin"

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
            --tx-in "$TXNID2"#1 \
            --tx-out=$OUT_ADDR+$(expr $LOVELACE1 - $FEE) \
            --tx-out=$OUT_ADDR+$LOVELACE2 \
         	--mint="$ASSET1" \
       	    --out-file $OUT_FILE
```

3. Generate a fee then build the transaction again after exporting the new fee varible

```
cardano-cli transaction calculate-min-fee \
--tx-body-file $OUT_FILE \
--tx-in-count 2 \
--tx-out-count 2 \
--witness-count 1 \
--testnet-magic $MAGICID \
--protocol-params-file protocol.json
```

export FEE=182265

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