#  TDEX Daemon
Daemon implementation to execute automated market marking strategies on top of TDEX

## Overview

The daemon exposes two HTTP/2 gRPC interfaces, one meant to be public to be consumed by traders that fully implements [BOTD #4](https://github.com/tdex-network/tdex-specs/blob/master/04-trade-protocol.md) called **trader interface** (by default on the port **9945**) and another private to be consumed by the liquidity provider for internal management called **operator interface** by default on the port **9000**). 

The daemon has an embedded Liquid wallet and sources blockchain information via a block explorer, at the time of writing, it supports only the [Blockstream fork of Electrs](https://github.com/blockstream/electrs). By default the daemon connects to [Blockstream.info](https://blockstream.info/liquid/api/)


**Operator API**

The API for the operator interface are documented [here](https://github.com/TDex-network/tdex-protobuf/blob/beta/docs/docs.md#operator)




## Data directory

The first time you run the daemon, it creates a **data directory** in `~/.tdex-daemon` and it is used to persist the wallet and the state in an embedded database. 
It's possible to use a different path for the data directory exporting the environment variable `TDEX_DATA_DIR_PATH`. If you use docker you must mount the volume pointing to the different chosen path.

**Be sure to replicate this data directory to keep your markets running in case of hardware failures. You can restore the access of your funds and the markets with your mnemonic seed**

## Run

Use one of the following methods to run a TDEX daemon on your machine:

* [Docker](#run-with-docker)
* [Standalone](#run-standalone)

## Run with Docker

#### Pull from Github Packages 

```sh
$ docker pull ghcr.io/tdex-network/tdexd:latest
```

#### Start the daemon

```sh
# Run on Liquid network connecting to blockstream.info for sourcing blockchain data
$ docker run -it -d --name tdexd --restart unless-stopped -p 9945:9945 -p 9000:9000 -v `pwd`/tdexd:/.tdex-daemon ghcr.io/tdex-network/tdexd:latest

# Run on Liquid connecting to a local explorer
$ docker run -it -d --name tdexd --restart unless-stopped -p 9945:9945 -p 9000:9000 -v `pwd`/tdexd:/.tdex-daemon -e TDEX_EXPLORER_ENDPOINT="http://127.0.0.1:3001" ghcr.io/tdex-network/tdexd:latest

# Run on Regtest connecting to a different explorer.
$ docker run -it -d --name tdexd --restart unless-stopped -p 9945:9945 -p 9000:9000 -v `pwd`/tdexd:/.tdex-daemon -e TDEX_NETWORK="regtest" -e TDEX_EXPLORER_ENDPOINT="http://127.0.0.1:3001"  ghcr.io/tdex-network/tdexd:latest

# Run on Liquid and specify USDt as base asset instead of default L-BTC
$ docker run -it -d --name tdexd --restart unless-stopped -p 9945:9945 -p 9000:9000 -v `pwd`/tdexd:/.tdex-daemon -e TDEX_BASE_ASSET="ce091c998b83c78bb71a632313ba3760f1763d9cfcffae02258ffa9865a37bd2" ghcr.io/tdex-network/tdexd:latest
```

This will mount the data direcory in a folder called `tdexd` in your current path.

See [Enviroment Variables](#environment-variables) for all available options. 

### Check the Logs

```sh
$ docker logs tdex
INFO[0000] trader interface is listening on :9945
INFO[0000] operator interface is listening on :9945
```

Now you are ready to [deposit funds](#deposit-funds) to create your fisrt market and start accepting incoming trades. 

## Run standalone


#### Install

1. [Download the latest release for MacOS or Linux](https://github.com/tdex-network/tdex-daemon/releases)

2. Move daemon and cli into a folder in your PATH (eg. `/usr/local/bin`) and rename the daemon as `tdexd` and the cli as `tdex`

3. Give executable permissions. (eg. `chmod a+x /usr/local/bin/tdexd` and `chmod a+x /usr/local/bin/tdex`)


#### Run

```sh
# Run on Liquid network connecting to blockstream.info for sourcing blockchain data
$ tdexd

# Run on Liquid connecting to a local explorer
$ export TDEX_EXPLORER_ENDPOINT="http://127.0.0.1:3001"
$ tdexd 

# Run on Regtest connecting to a different explorer.
$ export TDEX_NETWORK="regtest" 
$ export TDEX_EXPLORER_ENDPOINT="http://127.0.0.1:3001"
$ tdexd

# Run on Liquid and specify USDt as base asset instead of default L-BTC
$ export TDEX_BASE_ASSET="ce091c998b83c78bb71a632313ba3760f1763d9cfcffae02258ffa9865a37bd2"
$ tdexd
```

This will mount the data direcory in a folder called `.tdex-daemon` in your `$HOME`.

See [Enviroment Variables](#environment-variables) for all available options. 

Now you are ready to [deposit funds](#deposit-funds) to create your fisrt market and start accepting incoming trades. 

## Environment variables

TBD

## Deposit funds

To start a market, you need to deposit two reserves of two **Liquid assets** for the pair (called **Market**) you are providing liquidity for. Each **Market** has a BASE ASSET, wich is always the same per daemon, and a QUOTE ASSET.

To determine the spot price you can adopt different startegies, at the moment the supported one are **PLUGGABLE** and **BALANCED**. 

The PLUGGABLE strategy expects you to update the price manually, plugging in an external price feed that need to call the `UpdateMarketPrice` rpc method of the operator interface. 

The BALANCED strategy (this is the default when you create a market) uses **Automated Market Making** to determine the spot price. The initial ratio of the two amounts you deposit will represent the price of the first trade you accept in.
From that point on, the **automated market making strategy will self regulate the trading price**. It follows the *constant product market-making* formula. Every transaction that occurs on this market will adjust the prices of the market accordingly. It's a basic supply and demand automated market making system.


The following commands will uses the operator cli `tdex` to call the gRPC **operator** interface of `tdexd`. By default running on localhost on port 9000.


1. Create a new mnemonic seed (only the first time)

```sh
$ tdex genseed
```

2. Initialize the wallet (only the first time or after a restore from seed)

```sh
$ tdex init --seed="<generatedSeed>" --password <mypassword>
```

3. Unlock the wallet with chosen password

```sh
$ tdex unlock --password <mypassword>
```

4. Get a deposit address from the fee account 

```
$ tdex depositfee
```

Now send some L-BTC that will be used to subsidize liquid network fees. 

The CLI will create an ephemeral wallet with it to receive, will fragment them and send to the daemon. See [concurrent swap requests](#design.md) for the rationale behind.

5. Get a new deposit address from the MARKET ACCOUNT and send LBTC and another Liquid assets to automatically create and make a `market` tradable.

```sh
$ tdex depositmarket
```
Now send some base asset (by default is L-BTC) and quote asset of choice.

The CLI will create an ephemeral wallet with it to receive, will fragment them and send to the daemon. See [concurrent swap requests](#design.md) for the rationale behind.



## Manage markets

* Select the market

```sh
$ tdex market --base_asset=<BaseAssetHash> --quote_asset=<QuoteAssetHash>
```

Now the following commands will be launch against this market.

* Open the market using automated market making

```sh
$ tdex open
```
This makes the selected market availale for trading using the BALANCED market strategy 

* Close the market

```sh
$ tdex close
```

This makes the selected market NOT availale for trading.

* Change market making strategy to pluggable

```sh
$ tdex strategy --pluggable
```

* Update the price

```sh
$ tdex price --base_price=16000 --quote-price=0.001
```
This updates the current market price to be used for future trades.

* Open the market again

```sh
$ tdex open
```

