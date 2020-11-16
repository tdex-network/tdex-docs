# 🛠 TDEX SDK

JavaScript SDK for building trader-facing applications on top of TDEX

## ⬇️ Install

* Install with **yarn**

```sh
$ yarn add tdex-sdk@beta
```

* Install with **npm**

```sh
$ npm install --save tdex-sdk@beta
```

## 📄 Usage

### Trade

Trade against a Liquidity provider in the TDEX network. This fully implements [**BOTD#4**](https://github.com/tdex-network/tdex-specs/blob/master/04-trade-protocol.md)


#### With private key

```js
import { Trade, IdentityType, TradeType, fetchBalances } from 'tdex-sdk';

// Connect to specific provider and use Blockstream Esplora to source blockchain data.
// Change the providerUrl with the one you want to trade with. 
const trade = new Trade({
  providerUrl: 'provider.tdex.network:9945',
  explorerUrl: 'https://blockstream.info/liquid/api',
  identity: {
    chain: 'liquid', // or regtest
    type: IdentityType.PrivateKey,
    value: { 
      signingKeyWIF: "<WIF>",
      blindingKeyWIF: "<WIF>"
    },
  },
});

// Get a new address and his blinnding private key from identity interface
const {
  confidentialAddress,
  blindingPrivateKey
} = trade.identity.getNextAddress();

// Receiving Address and Change address are the same with Identity.PrivateKey
const changeAddrAndBlidning = trade.identity.getChangeAddress();


// Get the balances grouped by assetHash
const balances = fetchBalances(
  confidentialAddress,
  blindingPrivateKey,
  'https://blockstream.info/liquid/api'
);

// Asset hash of the market to trade
const LBTC = '6f0279e9ed041c3d710a9f57d0c02928416460c4b722ae3457a11eec381c526d';
const USDT = 'c5870288a7c9eb5db398a5b5e7221feb9753134439e8ed9f569b0eea5a423330';

//BUY = quote asset as input
// SELL = base asset as input
//
// If the type of the trade is BUY it means the base asset will be received by
// the trader.
//
// If the type of the trade is SELL it means the base asset will be sent by
// the trader.
const preview = await trade.preview({
  market,
  tradeType: TradeType.SELL,
  amount: 5000000,
});

console.log(preview)
/*
{
  "amountToBeSent": 5000000,
  "amountToReceive": 23869047,
  "amountToReceive": 8217213,
  "assetToBeSent": "6f0279e9ed041c3d710a9f57d0c02928416460c4b722ae3457a11eec381c526d",
  "assetToReceive": "c5870288a7c9eb5db398a5b5e7221feb9753134439e8ed9f569b0eea5a423330",
}
*/

// Sell some LBTCs
const txid = await trade.sell({
  market: {
    baseAsset: LBTC,
    quoteAsset: USDT,
  },
  amount: 100000,//satoshis unit
});

// Buy some LBTCs
const txid = await trade.buy({
  market: {
    baseAsset: LBTC,
    quoteAsset: USDT,
  },
  amount: 50000, //satoshis unit
});
```

#### With Mnemonic (HD Wallet) 

```js
const explorerUrl = "http://localhost:3001"

// Or Use HD wallet from mnemonic for both signign and blinding
const tradeWithMnemonic = new Trade({
  providerUrl: 'localhost:9945',
  explorerUrl: explorerUrl,
  identity: {
    chain: 'regtest', // or regtest
    type: IdentityType.Mnemonic,
    value: { 
      mnemonic:
      'mutuel ourson soupape vertu atelier dynastie silicium absolu océan légume skier',
      language: 'french', // optional
    },
    initializeFromRestorer: true // Scan the blockchain and restore previous addresses
    restorer: new EsploraIdentityRestorer(explorerUrl)
  },
});

// Wait for restore to be be completed. Can take a while.
try {
  await tradeWithMnemonic.isRestored();
} catch(e) {
  console.error(e);
}

// Now you can get addresses 
tradeWithMnemonic.identity.getNextAddress()
tradeWithMnemonic.identity.getChangeAddress()
```

### Identity

#### Send a confidential transaction with Mnemonic (HD Wallet) 

```js
import { walletFromAddresses, Wallet, fetchUtxos, Mnemonic, IdentityType } from 'tdex-sdk';

// Let's send to a confidential address a transaction on regtest

const explorerUrl = "http://localhost:3001"

// Create a Identity insatnce of type Mnemonic
const identity = new Mnemonic({
    chain: 'regtest', // or regtest
    type: IdentityType.Mnemonic,
    value: { 
      mnemonic:
      'mutuel ourson soupape vertu atelier dynastie silicium absolu océan légume skier',
      language: 'french', // optional
    },
    initializeFromRestorer: true // Scan the blockchain and restore previous addresses
    restorer: new EsploraIdentityRestorer(explorerUrl)
});

// Wait for restore to be be completed. Can take a while.
try {
  await identity.isRestored();
} catch(e) {
  console.error(e);
}

// First we create a Wallet instance using the local cache of the identity abstraction
const senderWallet = walletFromAddresses(identity.getAddresses(), 'regtest');

// then we fetch all utxos
const arrayOfArrayOfUtxos = await Promise.all(
  senderWallet.addresses.map((a) => fetchUtxos(wallet.address, explorerUrl))
);
// Flat them
const utxos = arrayOfArrayOfUtxos.flat();

// lets enrich them with confidential proofs using the prevout tx hexes
const txHexes = await Promise.all(
  utxos.map((utxo) => fetchTxHex(utxo.txid, explorerUrl))
);
const outputs = txHexes.map(
  (hex, index) => Transaction.fromHex(hex).outs[utxos[index].vout]
);
utxos.forEach((utxo, index) => {
  utxo.prevout = outputs[index];
});


console.log('Creating and blinding transaction...');
const tx = senderWallet.createTx();
const unsignedTx = senderWallet.buildTx(
  tx, // empty transaction
  utxos, // enriched unspents
  "el1qqgptwnszsecmr2klpvmqrmdmczd0gakldxef39425wtztfn7g3rsvpc8me5t8k0wkeaqh0nsnjlxd3kejtqdsln37tjrh9gvr", // recipient confidential address
  1000, // amount to be sent
  "5ac9f65c0efcc4775e0baec4ec03abdde22473cd3cf33c0419ca290e0751b225", // nigiri regtest LBTC asset hash
  identity.getNextChangeAddress() // change address we own
);

// Now we can sign with identity abstraction
const signedTx = await identity.signPset(unsignedTx);

// Finalize and extract tx to be a hex encoeded string ready for broadcast
const txHex = Wallet.toHex(signedTx);

```

### Swap

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
  psetBase64: "..."
})

//Bob parses the request and inspect the terms
let json = Swap.parse({
  message: swapRequestMessage,
  type: 'SwapRequest'
});

// Bob provides the transaction with his signed inputs and outputs
const swapAcceptMessage = swap.accept({
  message: swapRequestMessage,
  psetBase64: "..."
});


//Alice can parse again the message and inspect the terms (optional)
json = Swap.parse({
  message: swapAcceptMessage,
  type: 'SwapAccept'
});

// Alice adds his signed inputs to the transaction
const swapCompleteMessage = swap.complete({
  message: swapAcceptMessage,
  psetBase64: "..."
});

// Alice can sends the completed swap to Bob 
// Now Bob finalize the transaction and broadcast it 

```


### Wallet
