# 💻 TDEX CLI
Command line interface for making swaps and trades on TDEX

## Install

**Install from NPM**


```sh
# With Yarn
$ yarn global add tdex-cli
# With NPM
$ npm i -g tdex-cli
```

**Standalone binary (node/npm not needed)**


1. [Download the latest release for MacOS or Linux](https://github.com/tdex-network/tdex-cli/releases)

2. Move into a folder in your PATH (eg. `/usr/bin` or `/usr/local/bin`)

3. Give executable permission to it eg. `chmod a+x ./path/to/cli/tdex-cli`


By default, the `tdex-cli` will use the `~/.tdex` as data directory.

**Custom datadir (optional)**

Configure custom directory for data persistence. You should have write permissions. 

```sh
$ export TDEX_CLI_PATH=/path/to/data/dir 
$ tdex-cli help
```

## Commands


### Info

* Show current persisted state

```sh
$ tdex-cli info
```

### Network

* Set the network to work against 

> NOTICE With the --explorer flag you can set your own electrum REST server (Blockstream/electrs) for connecting to the blockchain.

```sh
# Mainnet
# This uses blockstream.info as explorer
$ tdex-cli network liquid
# Regtest
# This uses nigiri.network as explorer
$ tdex-cli network regtest
# Custom Esplora 
$ tdex-cli network regtest --explorer localhost:3001
```

### Wallet 

* Create or Restore Wallet

```sh
$ tdex-cli wallet
```

* Run again to print pubkey and address

```sh
$ tdex-cli wallet
```

* Get Wallet Balance

```sh
$ tdex-cli wallet balance
```

### Provider

* Select and connect to a liquidity provider

```sh
$ tdex-cli connect alpha-provider.tdex.network:9945
```
From this point, all the commands will work against this selected provider.


### Market

* List all available markets for current provider

```sh
$ tdex-cli market list
```

* Select a market to use for trading

```sh
$ tdex-cli market LBTC-USDt
```

* Get current exchange rate for selected market

```sh
$ tdex-cli market price
```

### Trade

* Start a swap against the selected provider

```sh
$ tdex-cli trade 
```


### Swap

* Create a swap request message

> NOTICE With the —-output flag you can customize the output file

```sh
$ tdex-cli swap request
```

* Import manually a SwapRequest and sign a resulting SwapAccept message

> NOTICE With the —-file flag you can customize the input file.

> NOTICE With the —-output flag you can customize the output file. By defualt the current directory of execution will be used.

```sh
$ tdex-cli swap accept
```

* Import a SwapAccept message and sign a resulting SwapComplete message 

> NOTICE With the —-file flag you can customize the input file.

> NOTICE With the —-output flag you can customize the output file. By defualt the current directory of execution will be used.

> NOTICE With the --push flag you can print the hex encoded extracted transaction and broadcast to the network.

```sh
$ tdex-cli swap complete
```

