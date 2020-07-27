#  TDEX Daemon
Daemon implementation to execute automated market marking strategies on top of TDEX

## Overview

The daemon exposes two HTTP/2 gRPC interfaces, one meant to be public to be consumed by traders that fully implements [BOTD #4](https://github.com/tdex-network/tdex-specs/blob/master/04-trade-protocol.md) called **trader interface** (by default on the port **9945**) and another private to be consumed by the liquidity provider for internal management called **operator interface** by default on the port **9000**).


The daemon has an embedded Liquid wallet and sources blockchain information via a block explorer, at the time of writing, it supports only the [Blockstream fork of Electrs](https://github.com/blockstream/electrs). By default the daemon connects to [Blockstream.info](https://blockstream.info/liquid/api/)


## Data directory

The first time you run the daemon, it creates a **data directory** in `~/.tdex-daemon` and it is used to persist the wallet and the state in an embedded database. 
It's possible to use a different path for the data directory exporting the environment variable `TDEX_DAEMON_PATH`. If you use docker you must mount the volume pointing to the different chosen path.

**Be sure to backup this data directory to keep your markets running in case of hardware failures and to being able to have access of your funds.**

## Run

Use one of the following methods to run a TDEX daemon on your machine:

* [Docker](#run-with-docker)
* [Standalone](#run-standalone)

## Run with Docker

#### Pull from Docker Hub 

```sh
$ docker pull truedex/tdex-daemon
```

#### Generate the wallet

This step is needed only the first time to create the data directory, generate and encrypt the wallet and initialize the database. In the following command example the data directory is mounted into the host's current working directory in a folder called `data` that will be created if not exists. Bu sure to have write permissions.

You can pass the [available options](#available-options) to configure the daemon.



```sh
# Run on Liquid network
$ docker run --rm -v `pwd`/data:/root/.tdex-daemon -it truedex/tdex-daemon

# Run on Liquid connecting to a different explorer
$ docker run --rm -v `pwd`/data:/root/.tdex-daemon -it truedex/tdex-daemon --explorer http://localhost:3000

# Run on Regtest connecting to a different explorer.
$ docker run --rm -v `pwd`/data:/root/.tdex-daemon -it truedex/tdex-daemon --regtest
--explorer http://localhost:3000

## Run on Liquid and specify a default provider fee
$ docker run --rm -v `pwd`/data:/root/.tdex-daemon -it truedex/tdex-daemon --fee 0.5
```

#### Encrypt with password

Once the command is launched you need to choose from the interactive shell `▸ Encrypted (AES-128-CBC)` and type in a password used to encrypt your wallet seed.


It should look like something like the following:

```sh
✔ A new wallet will be created and persisted in the chosen data directory. How do you want to store your seed? 🔑 · encrypted
✔ Type your password · ********

The mnemonic seed has been saved in /root/data/vault.json.
Be sure to make a safe backup of your data directory, it is the only way to restore your funds

You must restart the daemon exporting the env variable TDEX_PASSWORD
Shutting down...

```

The daemon will shut down itself and the container will be removed. Now it's time to run again, mapping the ports, mounting the data directory and passing the chosen password to let the daemon to automatically process incoming swaps. 

#### Start as daemon

> Put the right password instead of the `<ChosenPassword>` below.

```sh
$ docker run -d \
    --name tdex \
    --restart unless-stopped \
    -e TDEX_PASSWORD=<ChosenPassword> \
    -p 9945:9945 -p 9000:9000 \
    -v `pwd`/data:/root/.tdex-daemon \
    truedex/tdex-daemon
```
**NOTICE** You should run this command in case you are restoring from a backup of your data directory.

### Check the Logs

```sh
$ docker logs tdex
info: Trader gRPC server listening on 0.0.0.0:9945
info: Operator gRPC server listening on 0.0.0.0:9000
```


Now you are ready to [deposit funds](#deposit-funds) to create your fisrt market and start accepting incoming swaps. 

## Run standalone


#### Install

1. [Download the latest release for MacOS or Linux](https://github.com/tdex-network/tdex-daemon-alpha/releases)

2. Move into a folder in your PATH (eg. `/usr/bin` or `/usr/local/bin`) and rename it as `tdex-daemon`

3. Give executable permission to it eg. `chmod a+x /path/to/daemon/tdex-daemon`


#### Generate the wallet

```sh
# Run on Liquid network
$ tdex-daemon

# Run on Liquid connecting to a different explorer
$ tdex-daemon --explorer http://localhost:3000

# Run on Regtest connecting to a different explorer.
$ tdex-daemon --regtest --explorer http://localhost:3000

## Run on Liquid and specify a default provider fee
$ tdex-daemon --fee 0.5
```

#### Encrypt with password

```sh
✔ A new wallet will be created and persisted in the chosen data directory. How do you want to store your seed? 🔑 · encrypted
✔ Type your password · ********

The mnemonic seed has been saved in /root/data/vault.json.
Be sure to make a safe backup of your data directory, it is the only way to restore your funds

You must restart the daemon exporting the env variable TDEX_PASSWORD
Shutting down...
```


#### Start

> Put the right password instead of the `<ChosenPassword>` below.

```sh
$ export TDEX_PASSWORD=<ChosenPassword>
$ tdex-daemon
info: Trader gRPC server listening on 0.0.0.0:9945
info: Operator gRPC server listening on 0.0.0.0:9000
```
**NOTICE** You should run this command in case you are restoring from a backup of your data directory.


Now you are ready to [deposit funds](#deposit-funds) to create your fisrt market and start accepting trades. 


## Available options

```sh
Options:
  --help          Show help                                            
  --version       Show version number                                  
  --regtest, -r   Run in regtest mode                 
  --fee, -f       Specify a default fee to be used by markets
  --explorer, -e  Specify an Electrs HTTP REST endpoint                                         
```
> If a `config.json` file already exists in the chosen `datadir` given arguments will be discarded.



## Deposit funds

To start a market, you need to deposit two reserves of two **Liquid assets** for the **Market** you are providing liquidity for. 
The initial ratio of the two amounts you deposit will represent the price of the first trade you accept in.
From that point on, the **market making strategy will self regulate the trading price**. It follows the *constant product market-making*. In short, this model generates a full order-book based on an initial price for the market. Every transaction that occurs on this market will adjust the prices of the market accordingly. It's a basic supply and demand automated market making system. In the future, anyone could tweak and configure a different market making strategy.

1. Download and install the [`tdex-cli`](tdex-cli.md) 
2. Connect the CLI to the daemon via the gRPC **operator** interface. 
```sh
$ tdex-cli operator connect localhost:9000
```
> NOTICE: You need to deposit in a different address, called **FEE ACCOUNT**, an amount of LBTCs used by all markets to pay for transaction fees.

3. Get a deposit address from the FEE ACCOUNT and send some LBTC.
```
$ tdex-cli operator deposit --fee
```
4. Get a new deposit address from the MARKET ACCOUNT and send LBTC and another Liquid assets to automatically create and make a `market` tradable.
```sh
$ tdex-cli operator deposit
```
5. Profit! 
