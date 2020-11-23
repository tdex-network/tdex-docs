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

# Run on Regtest connecting to a local explorer and using regtest LBTC asset hash.
$ docker run -it -d --name tdexd --restart unless-stopped -p 9945:9945 -p 9000:9000 -v `pwd`/tdexd:/.tdex-daemon -e TDEX_NETWORK="regtest" -e TDEX_BASE_ASSET="5ac9f65c0efcc4775e0baec4ec03abdde22473cd3cf33c0419ca290e0751b225" -e TDEX_EXPLORER_ENDPOINT="http://127.0.0.1:3001"  ghcr.io/tdex-network/tdexd:latest

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

# Run on Regtest connecting to a local explorer and using regtest LBTC asset hash.
$ export TDEX_NETWORK="regtest" 
$ export TDEX_BASE_ASSET="5ac9f65c0efcc4775e0baec4ec03abdde22473cd3cf33c0419ca290e0751b225"
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

The list of avialble variables can be found [here](https://pkg.go.dev/github.com/tdex-network/tdex-daemon/config)

## Deposit funds

To start a market, you need to deposit two reserves of two **Liquid assets** for the pair (called **Market**) you are providing liquidity for. Each **Market** has a BASE ASSET, wich is always the same per daemon, and a QUOTE ASSET.

To determine the spot price you can adopt different startegies, at the moment the supported one are **PLUGGABLE** and **BALANCED**. 

The PLUGGABLE strategy expects you to update the price manually, plugging in an external price feed that need to call the `UpdateMarketPrice` rpc method of the operator interface. 

The BALANCED strategy (this is the default when you create a market) uses **Automated Market Making** to determine the spot price. The initial ratio of the two amounts you deposit will represent the price of the first trade you accept in.
From that point on, the **automated market making strategy will self regulate the trading price**. It follows the *constant product market-making* formula. Every transaction that occurs on this market will adjust the prices of the market accordingly. It's a basic supply and demand automated market making system.


The following commands will uses the operator cli `tdex` to call the gRPC **operator** interface of `tdexd`. By default running on localhost on port 9000.

* Initialize the local state of the CLI.


```sh
# By default it looks for the daemon operator gRPC interface on localhost:9000
$ tdex config init
```

* You can always check the current state with the following command

```sh
$ tdex config
```

* Create a new mnemonic seed (only the first time)

```sh
$ tdex genseed
```

* Initialize the wallet (only the first time or after a restore from seed)

```sh
$ tdex init --seed="<generatedSeed>" --password <mypassword>
```

* Unlock the wallet with chosen password

```sh
$ tdex unlock --password <mypassword>
```

* Get a deposit address from the fee account 

```
$ tdex depositfee
```

Now send some L-BTC that will be used to subsidize liquid network fees. 


* Get a new deposit address from the MARKET ACCOUNT and send LBTC and another Liquid assets to automatically create and make a `market` tradable.

```sh
$ tdex depositmarket
```
Now send some base asset (by default is L-BTC) and quote asset of choice.


* You can check the status of the market with the following command

```sh
$ tdex listmarket
```



## Manage markets

* Select the market

```sh
$ tdex config set base_asset=<BaseAssetHash> 
$ tdex config set quote_asset=<QuoteAssetHash>
```

You can always check the current state with the following command

```sh
$ tdex config
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

