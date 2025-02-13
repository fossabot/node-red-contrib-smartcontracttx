# node-red-contrib-smartcontracttx

<a href="https://stromdao.de/" target="_blank" title="STROMDAO - Digital Energy Infrastructure"><img src="./static/stromdao.png" align="right" height="85px" hspace="30px" vspace="30px"></a>

**Allows to run transactions on Ethereum based blockchains from within Node RED**

Call methods or run transactions using rpcUrl and ABI of existing contract. Allows OffChain DID/VP signing and verification.

[![npm](https://img.shields.io/npm/dt/node-red-contrib-smartcontracttx.svg)](https://www.npmjs.com/package/node-red-contrib-smartcontracttx)
[![npm](https://img.shields.io/npm/v/node-red-contrib-smartcontracttx.svg)](https://www.npmjs.com/package/node-red-contrib-smartcontracttx)
[![CO2Offset](https://api.corrently.io/v2.0/ghgmanage/statusimg?host=node-red-contrib-smartcontracttx&svg=1)](https://co2offset.io/badge.html?host=node-red-contrib-smartcontracttx)[![Code Quality](https://api.codiga.io/project/30556/score/svg)](https://app.codiga.io/public/project/30556/node-red-contrib-smartcontracttx/dashboard)[![CircleCI](https://circleci.com/gh/energychain/node-red-contrib-smartcontracttx/tree/main.svg?style=svg)](https://circleci.com/gh/energychain/node-red-contrib-smartcontracttx/tree/main)
[![FOSSA Status](https://app.fossa.com/api/projects/git%2Bgithub.com%2Fenergychain%2Fnode-red-contrib-smartcontracttx.svg?type=shield)](https://app.fossa.com/projects/git%2Bgithub.com%2Fenergychain%2Fnode-red-contrib-smartcontracttx?ref=badge_shield)

## Installation

Install using Node-RED Package Manager (Palett).

## Input - msg.payload

### OnChain - Transaction/Retrieve

In order to call a method of a SmartContract specify a JSON in `msg.payload` containing the method name as `method` and all required arguments in an array as `args`.

```javascript
{
  "method":"transfer",
  "args":["0x6B342cE1cb8671DDeeC57B62D78EB9333898d7da",20]
}
```

### OffChain - Present

To present a set of information (Object) signed to another party DID-VPs are used. In this case `msg.payload` must contain `presentTo` with the recipients publicKey (not address - [see here](https://ethereum.stackexchange.com/questions/13778/get-public-key-of-any-ethereum-account/79174) ) and `presentation` with the Data to crypt and sign.

```javascript
{
  "presentTo":"0x02eb74c1b28754e079ac138f0d1d73c0b9d82ba2a14ea3146f7f540e841ee43679",
  "presentation":{
    'Hello':'World',
    'TimeStamp': 1641166171599
  }
}
```

If `presentTo` is not specified the presentation itself will just be signed and not encrypted.

### Injection of unsecure values

If configuration option `Allow Insecure Inject` is set additional values might be specified in input `msg.payload` and will overwrite configured values:

```javascript
{
  ...
  "privateKey":"0x12356....",
  "contract":"0x123456...",
  "abi": [...],
  "rpcUrl": "https://integration.corrently.io/",
  ...
}
```

In case no privateKey is specified in either input msg.payload or configuration, a new privateKey gets generated.

## Output - msg.payload

For background compatibility all results are returned on `Output[0]`

`Output[1]` - OnChain Output. Returns results from method calls or transactions.

`Output[2]` - OffChain Output (JWT). Encrypted DID to be forwarded to other recipient.

`Output[3]` - OffChain Output. Presentations received and decoded.

## Cloudwallet support

Implementation allows to use [CloudWallet](https://rapidapi.com/stromdao-stromdao-default/api/cloudwallet) to persist received presentations/credentials or digital IDs via https://www.npmjs.com/package/cloudwallet

## Usage as Module (no Node-RED)

You might use this module without having Node-RED available. However this requires to *emulate* some of the stuff Node-RED typically provides.

### Installation

```
npm i --save node-red-contrib-smartcontracttx
```

### Hello World (DID)

```javascript
const SmartContractTX = require("node-red-contrib-smartcontracttx");

const app = async function() {
  const instanceAlice = new SmartContractTX();
  let msgAlice = {
    payload: {
        presentation: {
          'Hallo':'Welt'
        }
    }
  }

  // We need to do something with DIDComms (Message Alice wants to send) ...
  instanceAlice.setSender(async function(msgs) {
      // As soon as JWT got generated (Alice wants to send) - we forward to a new Bob
      const instanceBob = new SmartContractTX();
      let msgBob = msgs[2]; // Alice Output is [2] as we want the JWT
      console.log('Message from Alice to Bob',msgBob);
      // If Bob received a message he sends via output[3] to internal processes...
      instanceBob.setSender(async function(msgs) {
          console.log('Bob DID',msgs[3]);
      });
      await instanceBob.input(msgBob);
      console.log('End');
  });

  // Now we are ready to send Message from Alice
  await instanceAlice.input(msgAlice);
}

app();

```

## Tutorials / Usecases

- [Call Method on SmartContract - ERC20 totalSupply](https://github.com/energychain/node-red-contrib-smartcontracttx/blob/main/docs/UC1_Call_Method.md)
- [Issue Transaction](https://github.com/energychain/node-red-contrib-smartcontracttx/blob/main/docs/UC2_Transact_SC.md)
- [Basic Verifiable Presentation](https://github.com/energychain/node-red-contrib-smartcontracttx/blob/main/docs/UC3_VP_Offchain.md)
- [Digital Freddy - Digital Heritage](https://github.com/energychain/node-red-contrib-smartcontracttx/blob/main/docs/UC4_VP_DigitalFreddy.md)
- [Using Presentation Processing Services](https://github.com/energychain/node-red-contrib-smartcontracttx/blob/main/docs/CB_PresentationProcessingService.md)
- [Chinese whispers - Building a JWT/DID Bus](https://github.com/energychain/node-red-contrib-smartcontracttx/blob/main/docs/CB_Chinese_Whispers_Building_A_Bus.md)

## Maintainer / Imprint

<addr>
STROMDAO GmbH  <br/>
Gerhard Weiser Ring 29  <br/>
69256 Mauer  <br/>
Germany  <br/>
  <br/>
+49 6226 968 009 0  <br/>
  <br/>
kontakt@stromdao.com  <br/>
  <br/>
Handelsregister: HRB 728691 (Amtsgericht Mannheim)
</addr>

Project Website: https://co2offset.io/

## LICENSE
[Apache-2.0](./LICENSE)


[![FOSSA Status](https://app.fossa.com/api/projects/git%2Bgithub.com%2Fenergychain%2Fnode-red-contrib-smartcontracttx.svg?type=large)](https://app.fossa.com/projects/git%2Bgithub.com%2Fenergychain%2Fnode-red-contrib-smartcontracttx?ref=badge_large)