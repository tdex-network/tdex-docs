# 🛠 TDEX SDK


## JavaScript
JavaScript SDK for building trader-facing applications on top of TDEX

### ⬇️ Install

* Install with **yarn**

Not published yet.

* Install with **npm**

Not published yet.

### 📄 Usage

#### Trade

Trade against a Liquidity provider in the TDEX network. This fully implements [**BOTD#4**](https://github.com/tdex-network/tdex-specs/blob/master/04-trade-protocol.md)


```js
import { Trade } from 'tdex-sdk';

const trade = new Trade({
  chain: 'liquid',
  providerUrl: 'https://tdex.vulpem.com',
  explorerUrl: 'https://blockstream.info/liquid/api',
});

const LBTC = '6f0279e9ed041c3d710a9f57d0c02928416460c4b722ae3457a11eec381c526d';
const USDT = 'c5870288a7c9eb5db398a5b5e7221feb9753134439e8ed9f569b0eea5a423330';
const WIF = "...";
trade.buy({
  market: {
    baseAsset: LBTC,
    quoteAsset: USDT,
  },
  amount: 0.001,
  address: 'ex1q583qjfp8pd8wdxh6t6fc6cw536kt3l5t0lz2ua',
  privateKey: WIF
});


```

#### Swap

Create manually Swap messages without connecting to a provider. This fully implements [**BOTD#3**](https://github.com/tdex-network/tdex-specs/blob/master/03-swap-protocol.md)

```js
import { Swap } from 'tdex-sdk';

const swap = new Swap({ chain: "regtest" });

const LBTC = '5ac9f65c0efcc4775e0baec4ec03abdde22473cd3cf33c0419ca290e0751b225';
const USDT = 'c5870288a7c9eb5db398a5b5e7221feb9753134439e8ed9f569b0eea5a423330';

// Alice starts a swap proposal 
//
// You need to create and provide an unsigned transaction that has 
// enough inputs to cover amountToBeSent and the desired output
const swapRequestMessage = swap.request({
  assetToBeSent: USDT,
  amountToBeSent: 300,
  assetToReceive: LBTC,
  amountToReceive: 0.05,
  psbtBase64: "..."
})

//Bob parses the request and inspect the terms
const json = Swap.parse({
  message: swapRequestMessage,
  type: 'SwapRequest'
});

// Bob provides the transaction with his signed inputs and outputs
const swapAcceptMessage = swap.accept({
  message: swapRequestMessage,
  psbtBase64: "..."
});


//Alice can parse again the message and inspect the terms (optional)
const json = Swap.parse({
  message: swapAcceptMessage,
  type: 'SwapAccept'
});

// Alice adds his signed inputs to the transaction
const swapCompleteMessage = swap.complete({
  message: swapAcceptMessage,
  psbtBase64: "..."
});

// Alice can sends the completed swap to Bob 
// Now Bob finalize the transaction and broadcast it 

```
 
## Python

## Go 

## Rust