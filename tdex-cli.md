# TDEX CLI

Command line interface for making swaps and trades on TDEX

**⬇️ Install**

- Install with **yarn**

```sh
$ yarn global add tdex-cli@beta
```

- Install with **npm**

```sh
$ npm i -g tdex-cli@beta
```

By default, the `tdex-cli` will use the `~/.tdex-cli` as data directory, current state and private key will be stored in there.

**Custom datadir (optional)**

Configure custom directory for data persistence. You should have write permissions.

```sh
$ export TDEX_CLI_PATH=/absolute/path/to/data/dir
$ tdex-cli help
```

## Commands

### Info

- Show current persisted state

```sh
$ tdex-cli info
```

### Network

- Set the network to work against

> NOTICE With the --explorer flag you can set your own electrum REST server (Blockstream/electrs) for connecting to the blockchain.

```sh
# Mainnet
# This uses blockstream.info as explorer
$ tdex-cli network liquid
# Regtest
# This uses nigiri.network as explorer
$ tdex-cli network regtest
# Custom Esplora
$ tdex-cli network regtest --explorer http://localhost:3001
```

### Wallet

- Create or Restore Wallet

```sh
$ tdex-cli wallet init
```

- Generate a new address

```
$ tdex-cli wallet address
```

- Get Wallet Balance

```sh
$ tdex-cli wallet balance
```

- Send from Wallet

```sh
$ tdex-cli wallet send
```

### Provider

- Select and connect to a liquidity provider

```sh
$ tdex-cli connect https://provider.tdex.network:9945
```

From this point, all the commands will work against this selected provider.

### Market

- List all available markets for current provider

```sh
$ tdex-cli market list
```

- Select a market to use for trading

```sh
$ tdex-cli market LBTC-USDt
```

- Get current exchange rate for selected market

```sh
$ tdex-cli market price
```

### Trade

- Start a swap against the selected provider

```sh
$ tdex-cli trade
```
