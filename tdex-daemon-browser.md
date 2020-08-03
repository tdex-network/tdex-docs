# Browser support

The [Daemon](tdex-daemon.md) and the [Javascript SDK](tdex-sdk.md) both support [gRPC-Web](https://github.com/grpc/grpc-web) out-of-the-box which is a cutting-edge spec that enables invoking gRPC services from modern browsers.

In order to that, gRPC-web clients connect to gRPC services via a special proxy. You can find a dockerized version [here](https://github.com/TDex-network/docker-grpcwebproxy)

## Run

Use one of the following methods to run a TDEX daemon with gRPCWeb support on your machine:


### Run with docker compose


1) If it's the first time (ie. you need to create a wallet). If not, skip to the following step

```
# Run on Liquid network and select Encrypted
$ docker run --rm -v `pwd`/data:/root/.tdex-daemon -it truedex/tdex-daemon

```

2) Clone the repository

```sh
$ git clone https://github.com/TDex-network/tdex-daemon-box
```

3) Move the `docker-compose.yml` and `Caddyfile` into the same directory where `data` (created at step 1) is stored

```
$ export TDEX_PASSWORD=MyPassword 
$ docker-compose up -d
```


### Run standalone

- Start tdex-daemon `tdex-daemon`
- Download pre-built binary of grpcwebproxy from [here](https://github.com/improbable-eng/grpc-web/releases)
- Start gowebproxy `grpcwebproxy --backend_addr=localhost:9945 --run_tls_server=false --allow_all_origins`

