# How to program an upgradable smart contract step by step

## introduction 

As an information transform network, it is good for internet. And as we known [Agile](http://agilemanifesto.org/) and iteration is becoming popular in software development field.

The good news for blockchain is that blockchain is design to [Tamper resistance](https://en.wikipedia.org/wiki/Tamper_resistance), but following is the bad news, it is hard to upgrade in blockchain field (e.g even we have found bugs in smart contract, it is hard to upgrade/fix it), which is preventing blockchain evolving.

We would introduce a way to upgrade your smart contract with [ZepplinOS](https://zeppelinos.org/).

## installation/preparation

First of all, we need to setup/prepare environment. Here we need: 

- zos
- ganache-cli
- truffle

### install zos

We will use command line tool for ZeppelinOS `zos` :

```
$ npm install -g zos
```

### install truffle/ganache-cli

Install truffle, you can refer the [truffle official website](https://truffleframework.com/docs/getting_started/installation).

```
$ npm install -g truffle
```

Install ganache-cli

```
$ npm install --save ganache-cli
```

## init project

```
$ mkdir zos-demo
$ cd zos-demo
$ npm init -y
$ zos init zos-demo
```

**Notice**: the zos.json file that was produced. This is the configuration file that makes zos aware of your smart contract architecture. Feel free to read up on the zos.json file format to understand how zos sees your project.

```
$ npm install --save zos-lib
```

## prepare smart contract in solidity

In `contracts/` folder create a new file named `CounterContrat.sol` with following code:

```
pragma solidity ^0.4.21;
import "zos-lib/contracts/migrations/Migratable.sol";
contract CounterContract is Migratable {
 uint256 public counter;
  function initialize(uint256 _counter) isInitializer("CounterContract", "0") public {
   counter = _counter;
 }

 function increment() public {
   counter += 1; 
 }
}
```

For an upgradable contract, instead, replace constructor with an `initialize` function to set up the initial smart contract state. Make sure the `initialize` function with `isInitializer` modifer that take the name of your contract and version ID.

**NOTE:** If you do use a constructor in an upgradeable smart contract, everything you set in the constructor will be ignored.

### Test locally (start ganache-cli)

Issue following command in a separate console:

```
$ npx ganache-cli --port 9545
```

Then deploy smart contract to local network. (in another console) and add contract to zos and push it to local network.

```
$ zos add CounterContract
$ zos push --network local
```

**NOTE:** If encountering following failure, check your `hosts` file to make sure define `localhost` as '127.0.0.1'

```
Writing artifacts to ./build/contracts

Could not connect to your Ethereum client. Please check that your Ethereum client:
    - is running
    - is accepting RPC connections (i.e., "--rpc" option is used in geth)
    - is accessible over the network
    - is properly configured in your Truffle configuration file (truffle.js)
```

Mostly it should treat `localhost` as '127.0.0.1', but sometime `hosts` file corrupts.

You can edit `truffle-config.js` to change host to '127.0.0.1'.

### invoke contract

Next, we’ll create and initialize an upgradeable instance of CounterContract with zos. The `--args` option corresponds with the parameters of our initialize method.

```
$ zos create CounterContract --init initialize --args 42 --network local
```
**Note:** the output line ‘CounterContract proxy: <address>’. This is the address you will use for CounterContract. Save this address, as it is the permanent address for CounterContract. As following:

```
Creating CounterContract proxy and calling initialize with:
 - _counter (uint256): "42"
 TX receipt received: 0x18746572bd744e231d8f0a9996a9afba8c2424e9c6329d1b7e71d429926b31d7
 CounterContract proxy: 0xfb011a654e7c39a19932ced52c04adec748ecdff
Successfully written zos.local.json
0xfb011a654e7c39a19932ced52c04adec748ecdff
```

Then we can test it in truffle console, launch truffle console by:

```
$ npm truffle console --network=local
```

We will see console with prefix `truffle(local)>`. And issue following command to set var:

```
truffle(local)> counterContract = CounterContract.at("0xfb011a654e7c39a19932ced52c04adec748ecdff")
truffle(local)> counterContract.increment()
truffle(local)> counterContract.counter().then(counter => counter.toNumber())
truffle(local)> 43

```

## upgrade smart contracts

**NOTE:** for upgradable smart contracts, only support to add varables/functions, must not remove codes from previous contracts.

For example, we add a new function as following:

```
pragma solidity ^0.4.21;
import "zos-lib/contracts/migrations/Migratable.sol";

contract CounterContract is Migratable {
 uint256 public counter;
mapping(uint256 => address) public history;

function initialize(uint256 _counter) isInitializer("CounterContract", "0") public {
   counter = _counter;
 }

function increment() public {
   counter += 1; 
   history[counter] = msg.sender;
 }

function incrementByTwo() public {
   counter += 2;
   history[counter] = msg.sender;
 }
}
```

### push and update contracts

exit truffle console with command (.exit)

```
$ zos push --network local
$ zos update CounterContract --network local
$ npx truffle console --network=local 
```

Entering truffle console, 

```
truffle(local)> counterContract = CounterContract.at("0xfb011a654e7c39a19932ced52c04adec748ecdff")
truffle(local)> counterContract.incrementByTwo()
truffle(local)> counterContract.counter().then(counter => counter.toNumber())
truffle(local)> 45

```

**NOTE:** If at any point you’ve stopped Ganache and need to restart this process all over, make sure you delete zos.local.json file as well. This isn’t a problem for other networks, since typically networks don’t get wiped out :)

Edited from [ZeppelinOS Blog](https://blog.zeppelinos.org/getting-started-with-zeppelinos/)
