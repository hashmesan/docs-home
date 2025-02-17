---
description: Interacting With MetaMask
---

# Interacting With Metamask

{% hint style="info" %}
For instructions on how to install and setup Metamask to work with Harmony blockchain please click [here](../../../network/wallets/browser-extensions-wallets/metamask-wallet.md).
{% endhint %}

You can connect and sign transactions with Metamask using the Web3 library - which is fully compatible with the Harmony RPC API.

Do not forget that an important difference from One Wallet is that Metamsk sends transactions itself (extension side). While One Wallet only sign transaction, and then sending happens on browser side.

To use next example, you need to include the following libraries:

```
npm i '@metamask/detect-provider' --save
npm i web3 --save
npm i 'bn.js' --save
```

First step - you need to detect provider and connect to Metamask:&#x20;

```javascript
import detectEthereumProvider from '@metamask/detect-provider';

let ethAddress;
let isAuthorised = false;

const handleAccountsChanged = (accounts) => {
  if (accounts.length === 0) {
    console.error('Not found accounts');
  } else {
    ethAddress = accounts[0];
    
    console.log('Your address: ', ethAddress);
  }
}

export const signInMetamask = async () => {
    const provider = await detectEthereumProvider();

    // @ts-ignore
    if (provider !== window.ethereum) {
      console.error('Do you have multiple wallets installed?');
    }

    if (!provider) {
      console.error('Metamask not found');
      return;
    }

    // MetaMask events
    provider.on('accountsChanged', handleAccountsChanged);

    provider.on('disconnect', () => {
      console.log('disconnect');
      isAuthorised = false;
    });
    
    provider.on('chainIdChanged', chainId => console.log('chainIdChanged', chainId));

    provider
      .request({ method: 'eth_requestAccounts' })
      .then(async params => {
        handleAccountsChanged(params);
        isAuthorised = true;
      })
      .catch(err => {
        isAuthorised = false;
        
        if (err.code === 4001) {
          console.error('Please connect to MetaMask.');
        } else {
          console.error(err);
        }
      });
}
```

Next step -  you can use connected provider with Web3 to sign and send transactions:

```javascript
new Web3(window.web3.currentProvider) 

/* provider will use network RPC, wich was selected in MetaMask */
```

```javascript
const accounts = await ethereum.enable();
    
/* Now any request to sign a transaction will be redirected to MetaMask */
```

```javascript
import Web3 from 'web3';
const BN = require('bn.js');

const sendTransaction = async () => {
    const web3 = new Web3(window.ethereum);
    
    const receiverAddress = '0x430506383F1Ac31F5FdF5b49ADb77faC604657B2';
    
    const gas = 6721900;
    const gasPrice = new BN(await web3.eth.getGasPrice()).mul(new BN(1));
    
    const result = await web3.eth
      .sendTransaction({
        from: account,
        to: receiverAddress,
        value: 1 * 1e18, // 1ONE
        gasPrice,
        gas,
      })
      .on('error', console.error);

    console.log(`Send tx: ${result.transactionHash} result: `, result.status);
}
```

After executing this function, an interactive MetaMask window will open in which you can sign the transaction and change the gasPrice / gasLimit parameters - if it necessary.

Full code example will be here: [https://github.com/harmony-one/ethhmy-bridge.frontend/blob/web3\_hmy/src/pages/Examples/TransactionExample.tsx](https://github.com/harmony-one/ethhmy-bridge.frontend/blob/web3\_hmy/src/pages/Examples/TransactionExample.tsx)

Also you can use all Provider API from official MetaMask and Web3 docs: [https://docs.metamask.io/guide/ethereum-provider.html#table-of-contents](https://docs.metamask.io/guide/ethereum-provider.html#table-of-contents)
