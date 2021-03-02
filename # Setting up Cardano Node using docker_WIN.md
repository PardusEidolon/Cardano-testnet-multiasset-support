# Setting up Cardano Node using docker for Windows

1. First download and install docker: https://www.docker.com/products/docker-desktop
2. Create a cardano node folder on your main drive
   1. inside create the subfolders "mary-data" & "mary-ipc"
3. Run the **cardano-node** below with the appropriate paths.

```
docker run --detach --name=relay1 -v C:\docker\cardano-node\mary-data:/data/db -v C:\docker\cardano-node\mary-ipc:/ipc -e NETWORK=testnet inputoutput/cardano-node run
```
4. Now that we have an image all set up and running export the cardano socket.
```
export CARDANO_NODE_SOCKET_PATH
```

5. Next run this line of code to query the tip of the node.

```
cardano-cli query tip --testnet-magic 1097911063
```