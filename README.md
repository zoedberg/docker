# Docker containers for LNP/BP daemons and tools

This containers are maintained by LNP/BP Standards Association

- **Bitcoin Core**: <https://hub.docker.com/r/lnpbp/bitcoind>
  Minimalistic build without wallet (but with ZMQ) on Debian Buster
- **Bitcoin Core** for SigNet: <https://hub.docker.com/r/lnpbp/bitcoind-signet>
  Minimalistic build of the original SigNet fork without wallet (but with ZMQ)
  on Debian Buster
- **c-Lightning**: <https://hub.docker.com/r/lnpbp/lightningd>
  Standard c-lightning build
- **Electrs**:
  Rust implementation of the Electrum server

## Quickstart

1. Clone this repository and change to its dir: 
    ```shell script
   git clone https://github.com/lnp-bp/docker
   cd docker
    ```
2. Initialize volume and default config file with this commands:
    ```shell script
    docker volume create blockchain
    docker run --rm -v $PWD:/source -v blockchain:/dest -w /source scratch cp ./bitcoin.conf /dest
    ```
2. Go to the `mainnet`, `signet` or `liquid` directory and perform 
   `docker-compose up` command

Now you will have bitcoind running for mainnet and signet with JSON-RPC interface opened for the local machine.
You will be able to access it with the user name `bitcoin` and password `bitcoin`.

To modify the default JSON-RPC user name, password, IP addresses and other bitcoind options edit the `bitcoin.conf`
file in the root directory of the repository and each time perform this command:
```shell script
docker-compose down
docker run --rm -v $PWD:/source -v blockchain:/dest -w /source scratch cp ./bitcoin.conf /dest
docker-compose up
```

## Changing blockchain directory location

You can use your axisting bitcoin blockchain directory by first creating docker volume pointing to it
with
```shell script
docker volume create --driver local --opt o=bind --opt type=none --opt device=/var/lib/bitcoin bitcoin 
```
command, edit `external.env` file paths and then by starting docker-compose with `--env=external.env` option

### Editing compose file

Replace all occurrences of `blockchain` string within the volumes block with the desired destination directory. 
To locate volumes look for the following pattern:
```yaml
    volumes:
      - "blockchain:/var/lib/bitcoind"
```
so it will become, for example
```yaml
    volumes:
      - "/Volumes/ExternalDrive/bitcoin:/var/lib/bitcoind"
```

In this case you will need to copy `bitcoin.conf` file to that destination; otherwise bitcoind will run with the
default parameters

## Creating container from the command line

Execute the following command:
```shell script
docker run \
  -p 8332:8332 -p 8333:8333 \
  -v /Volumes/ExternalDrive/bitcoin:/var/lib/bitcoind \
  --name bitcoind \
  lnpbp/bitcoind:latest
```
 with replacing "/Volumes/ExternalDrive/bitcoin" with the desired location for the blockchain data.
 
 You do not need to clone this git repository to execute this command, however you will need to copy `bitcoin.conf` file 
 to that destination beforehand; otherwise bitcoind will run with the default parameters


## Using command-line tools

The most simple way of using tools is to create aliases:
```shell script
alias bitcoin-cli='docker exec bitcoind-mainnet bitcoin-cli -rpcpassword=bitcoin -rpcuser=bitcoin'
alias lightning-cli='docker exec lightningd-mainnet lightning-cli --mainnet --lightning-dir /var/lib/lightningd'
alias liquid-cli='docker exec elementsd-liquidv1 elements-cli -chain=liquidv1 -rpccookiefile=/var/lib/elementsd/liquidv1/.cookie'
alias signet-cli='docker exec bitcoind-signet bitcoin-cli --signet -rpcpassword=bitcoin -rpcuser=bitcoin'
alias sightning-cli='docker exec lightningd-signet lightning-cli --signet --lightning-dir /var/lib/lightningd'
```
