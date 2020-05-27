# 💸 TDEX Daemon
Daemon implementation to execute automated market marking strategies on top of TDEX

## Install

Use one of the following methods to run the daemon, either with *docker* or *standalone*. The daemon will expose two HTTP/2 gRPC interfaces, one meant to be public to be consumed by traders that fully implements [BOTD #4](https://github.com/Sevenlab/tdex-specs/blob/master/04-trade-protocol.md) called **trader interface** on the canonical port **9945** and another private to be consumed by the liquidity provider for internal management called **operator interface** on the port **9000**.

### Docker

#### Pull from Docker Hub 

```sh
$ docker pull sevenlab/tdex-daemon
```

#### Run the container

```sh
$ docker run -p 9945:9945 -p 9000:9000 -v /data:/root/.tdex-daemon -it sevenlab/tdex-daemon --regtest
✔ How do you want to store your seed? 🔑 · plain
info: Trader gRPC server listening on 0.0.0.0:9945
info: Operator gRPC server listening on 0.0.0.0:9000
```
To detach the tty without exiting the shell, use the escape sequence Ctrl+P followed by Ctrl+Q

See [Usage](#usage) to understand more about `Data directory` and how to encrypt the daemon's `wallet`.
See [Available options](#available-options) to configure the daemon.

### Standalone

#### Download standalone binary (node/npm not needed)

* [Download latest release for MacOS](https://tdex-builds.s3-eu-west-1.amazonaws.com/daemon/darwin/tdex-daemon)
* [Download latest release for Linux amd64](https://tdex-builds.s3-eu-west-1.amazonaws.com/daemon/linux/tdex-daemon)

Move into a folder in your PATH (eg. `/usr/bin` or `/usr/local/bin`)

#### Run the binary

```sh
$ tdex-daemon --regtest
✔ How do you want to store your seed? 🔑 · plain
info: Trader gRPC server listening on 0.0.0.0:9945
info: Operator gRPC server listening on 0.0.0.0:9000
```

See [Usage](#usage) to understand more about `Data directory` and how to encrypt the daemon's `wallet`.
See [Available options](#available-options) to configure the daemon.


## Usage

#### Data Directory

Once the daemon is launched it will create a data directory `~/.tdex-daemon` containing the default configuration file `config.json`. It's possible to use a different path for the data directory with the environment variable `TDEX_DAEMON_PATH`.

#### Wallet

It will be created a wallet for the daemon and stored in the chosen data directory in a file called `vault.json`.
You can encrypt it with a password and if you decide to do so the daemon will save it encrypted and shutdown.
Then start again exporting the environment variable `TDEX_PASSWORD` with the chosen password so the daemon could automatically process incoming swap requests. 
> DO NOT FORGET THE PASSWORD, OR YOU WILL NOT BE ABLE TO RECOVER YOUR FUNDS

**Docker**

```sh
$ docker run --rm -v /data:/root/.tdex-daemon -it sevenlab/tdex-daemon --regtest 
✔ How do you want to store your seed? 🔑 · encrypted
✔ Type your password · ********
Wallet created! Restart the daemon exporting the env variable TDEX_PASSWORD
Shutting down...
```

Restart again passing the env variable `TDEX_PASSWORD`

```sh
$ docker run -d \
    --name tdex \
    --restart unless-stopped \
    -e TDEX_PASSWORD=ChosenPassword \
    -p 9945:9945 -p 9000:9000 \
    -v $(pwd)/data:/root/.tdex-daemon \
    sevenlab/tdex-daemon
751266e0ff48fad8c5af15ec5fc6f2678237429e4d7b2a4f59eb8001fa6bcbfa
```

Check the logs

```sh
$ docker logs tdex
warn: Configuration file already exists at path /root/.tdex-daemon. Given arguments will be discarded
info: Trader gRPC server listening on 0.0.0.0:9945
info: Operator gRPC server listening on 0.0.0.0:9000
```

**Standalone**

```sh
$ tdex-daemon --regtest
✔ How do you want to store your seed? 🔑 · encrypted
✔ Type your password · ********
Wallet created! Restart the daemon exporting the env variable TDEX_PASSWORD
Shutting down...
$ export TDEX_PASSWORD=ChosenPassword
$ tdex-daemon
info: Trader gRPC server listening on 0.0.0.0:9945
info: Operator gRPC server listening on 0.0.0.0:9000
```

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

To start a market, you need to deposit two reserves for the **Market** you are providing liquidity for. 
The initial ratio of two amounts you deposit will represent the starting price you give to that pair. 
From that point on, the **market making strategy will self regulate the trading price**.

> NOTICE: You will also need to deposit in a different address, called **FEE ACCOUNT**, an amount of LBTCs used by all markets to pay for transaction fees.

1. Download and install the [`tdex-cli`](tdex-cli.md) 
2. Connect the CLI to the daemon with the gRPC **operator** interface. 
```sh
$ tdex-cli operator connect localhost:9000
```
3. Get the deposit address from the FEE ACCOUNT and send some L-BTC
```
$ tdex-cli operator deposit --fee
```
4. Get a new deposit address from the MARKET ACCOUNT and send L-BTC and other Liquid assets to create and start a `market`
```sh
$ tdex-cli operator deposit
```
5. Profit! 



