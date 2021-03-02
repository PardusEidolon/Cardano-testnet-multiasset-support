# How to use Multi-Asset Tokens in the Cardano Node Testnet with docker on windows
---

## Setting up and running a cardano node on the mary testnet

1. First download and install docker: https://www.docker.com/products/docker-desktop
2. Create a cardano node folder on your main drive
    - *Inside create the subfolders "mary-data" & "mary-ipc" I just happen to have made a directory on my local C:\ drive*
3. Run the **cardano-node** below with the appropriate paths.
   - *you can name your image whatever you want to be honest*

```
docker run --detach --name=relay1 -v C:\docker\cardano-node\mary-data:/data/db -v C:\docker\cardano-node\mary-ipc:/ipc -e NETWORK=testnet inputoutput/cardano-node run
```
1. Now that we have an image all set up and running export the cardano socket.
```
export CARDANO_NODE_SOCKET_PATH
```

5. Next run this line of code to query the tip of the node.
      - *To find the magic number, there should be a file located in your mar-ipc folder, alternativly you can find the magic number here https://hydra.iohk.io/build/5367762/download/1/index.html just select the testnet shellygenesis and look for "networkMagic"*

```
cardano-cli query tip --testnet-magic 1097911063
```