[back](./README.md)
# Contracts - intended usage

## Introduction
You interact with a Aeternity node through HTTP.
To learn more about contracts and contract life cycles see [the doc](/contracts/contracts.md).

There are two basic types of API calls, off-chain operations and on-chain transactions.

For an up to date list of all HTTP API endpoints and their arguments see
[Epoch HTTP API](https://aeternity.github.io/epoch-api-docs/?config=https://raw.githubusercontent.com/aeternity/epoch/master/apps/aehttp/priv/swagger.json)

## Sophia calldata creation

A number of functions described below create calldata for contract call or
contract create transactions ([code/encode-calldata](#encode-call-data),
[code/call](#call), [create/compute](#contract-create),
[call/compute](#contract-call)). These support two ways of specifying the
function and arguments to the call:
- **Unchecked (legacy)**: Function name (except for `create/compute`) and arguments
  as a Sophia constant tuple. No checks are made to ensure that the given
  arguments match what the contracts expects.
- **Checked**: An argument `call` containing Sophia source code for a contract
  snippet containing (at least) a prototype for the function to be called and a
  special function `__call()` whose body is a single call to this function. This
  contract is type checked and the given arguments are checked to be compatible
  with the type that the called contract expects.

  For example, to call a function `swap` that expects a record argument you can
  give the following `call`:
  ```
  contract CallExample =
    record pt = {x : int, y : int}
    function swap : pt => pt
    function __call() = swap({x = 3, y = 4})
  ```
  The call contract can contain more definitions than are needed to type check
  the call. In particular, it can be the complete source code of the contract
  to be called, with an added `__call` function.

## Off chain operations

An epoch node provides some utility functions to help you create contract transactions.


### Contract Compile
[/debug/contracts/code/compile doc](https://aeternity.github.io/epoch-api-docs/?config=https://raw.githubusercontent.com/aeternity/epoch/master/apps/aehttp/priv/swagger.json#/internal/CompileContract)

### Encode Call Data
[/debug/contracts/code/encode-calldata doc](https://aeternity.github.io/epoch-api-docs/?config=https://raw.githubusercontent.com/aeternity/epoch/master/apps/aehttp/priv/swagger.json#/internal/EncodeCalldata)

### Call
[/debug/contracts/code/call doc](https://aeternity.github.io/epoch-api-docs/?config=https://raw.githubusercontent.com/aeternity/epoch/master/apps/aehttp/priv/swagger.json#/internal/CallContract)

The arguments to the /debug/contracts/code/call endpoint are:
* `code` - either the byte code of the contract to call (if abi is either "sophia" or "evm") or an address of a contract on chain (if abi is "sophia-address").
* `call` - the call contract as described [above](#sophia-calldata-creation) (if abi is "sophia" or "sophia-address")
* `function` (legacy) - the name of the function in the contract to call (if abi is "sophia" or "sophia-address").
* `arg` - the argument to the call as Sophia constants (legacy, when abi is "sophia" or "sophia-address") or as an solidity encoded call-data (if abi is "evm").
* `abi` - one of "sophia", "sophia-address" or "evm".

### Decode Call Data
[/debug/contracts/code/decode-data doc](https://aeternity.github.io/epoch-api-docs/?config=https://raw.githubusercontent.com/aeternity/epoch/master/apps/aehttp/priv/swagger.json#/internal/DecodeData)

## On chain transactions

An epoch node provides some APIs to format contract transactions and an API for submitting a signed transaction.

There are two contract transactions available: create and call.

You:
* Format the contract transaction using the corresponding API;
* (As for any other transaction) sign offline (i.e. outside of the epoch node) the transaction according to consensus;
* (As for any other transaction) submit the transaction to the epoch node using [its API](https://aeternity.github.io/epoch-api-docs/?config=https://raw.githubusercontent.com/aeternity/epoch/master/apps/aehttp/priv/swagger.json#/external/PostTransaction).

In order to affect the state of the chain you have to submit the signed transaction to a mining node.

### Contract Create
[/debug/contracts/create doc](https://aeternity.github.io/epoch-api-docs/?config=https://raw.githubusercontent.com/aeternity/epoch/master/apps/aehttp/priv/swagger.json#/internal/PostContractCreate)

[/debug/contracts/create/compute doc](https://aeternity.github.io/epoch-api-docs/?config=https://raw.githubusercontent.com/aeternity/epoch/master/apps/aehttp/priv/swagger.json#/internal/PostContractCreateCompute)

### Contract Call
[/debug/contracts/call doc](https://aeternity.github.io/epoch-api-docs/?config=https://raw.githubusercontent.com/aeternity/epoch/master/apps/aehttp/priv/swagger.json#/internal/PostContractCall)

[/debug/contracts/call/compute doc](https://aeternity.github.io/epoch-api-docs/?config=https://raw.githubusercontent.com/aeternity/epoch/master/apps/aehttp/priv/swagger.json#/internal/PostContractCallCompute)

## Reading chain data

### Contract Call Result
[/transactions/{hash}/info doc](https://aeternity.github.io/epoch-api-docs/?config=https://raw.githubusercontent.com/aeternity/epoch/master/apps/aehttp/priv/swagger.json#/external/GetTransactionInfoByHash)

