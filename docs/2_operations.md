---
slug: /operations
sidebar_position: 2
---

# Operation Type

ERC-5189 transactions are called "operations". They are composed of information needed to execute the transaction on-chain and information needed to validate the transaction off-chain. ERC-5189 operations are called "operations" because they don't necessarily represent a single transaction or a single user; they represent any action that can be performed on-chain while compensating the relayer.


| Field                      | Type    | Description                                                                                                                                                 |
| -------------------------- | ------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| entrypoint                 | address | Contract address that must be called with `callData` to execute the `operation`.                                                                            |
| callData                   | bytes   | Data that must be passed to the `entrypoint` call to execute the `operation`.                                                                               |
| fixedGas                   | uint64  | Amount of gas that the operation will pay for, regardless execution costs, and independent from `gasLimit`.                                                 |
| fixedGas                   | uint256 | Fixed amount of gas units that the operation is expected to pay, regardless from the used gas limit. |
| gasLimit                   | uint64  | Minimum gasLimit that must be passed when executing the `operation`.                                                                                        |
| feeToken                   | address | Contract address of the token used to repay the bundler. _(`address(0)` for the native token)_.                                                             |
| endorser                   | address | Address of the endorser contract that should be used to validate the `operation`.                                                                           |
| endorserCallData           | bytes   | Additional data that must be passed to the `endorser` when calling `isOperationReady()`.                                                                    |
| endorserGasLimit           | uint64  | Amount of gas that should be passed to the endorser when validating the `operation`.                                                                        |
| maxFeePerGas               | uint256 | Max amount of basefee that the `operation` execution is expected to pay. _(Similar to [EIP-1559](./eip-1559.md) `max_fee_per_gas`)_.                        |
| priorityFeePerGas          | uint256 | Fixed amount of fees that the `operation` execution is expected to pay to the bundler. _(Similar to [EIP-1559](./eip-1559.md) `max_priority_fee_per_gas`)_. |
| feeScalingFactor           | uint256 | Scaling factor to convert the computed fee into the `feeToken` unit.                                                                                        |
| feeNormalizationFactor     | uint256 | Normalization factor to convert the computed fee into the `feeToken` unit.                                                                                  |
| hasUntrustedContext        | bool    | If `true`, the operation _may_ have untrusted code paths. These should be treated differently by the bundler (see untrusted environment).                   |
| chainId                    | uint256 | Chain ID of the network where the `operation` is intended to be executed.                                                                                   |

It is important to note that the only fields that are required to reach the chain are `entrypoint` and `callData`. The rest of the fields are used to validate the operation off-chain and to calculate the fees that the operation will pay. This means that it is not possible to compute the operation hash from within the EVM, as the operation hash is a hash of the operation's off-chain representation.

The advantage of this approach is that it maintains the gas costs of the operation as low as possible, as most of the fields are ephemeral. It also allows for easier versioning of the standard, as new fields can be added without breaking existing implementations.

## Operation Hash

The operation hash is the CIDv1 IPFS multihash of a `raw` file, containing the canonical JSON representation of the operation. Using canonical JSON representation is important to ensure that the hash is fully deterministic across all parties; and the use of IPFS allows for the operation for easy reverse lookups.

To better illustrate this, here is an example of an operation hash:

[ipfs://bafkreiciblpbltjirn3nfuicwnkphizae5u2acjjq72g44uw2ed55gkja4/](ipfs://bafkreiciblpbltjirn3nfuicwnkphizae5u2acjjq72g44uw2ed55gkja4/)

By looking up the contents of this IPFS hash, you will find the JSON representation of the operation.

```
ipfs cat bafkreiciblpbltjirn3nfuicwnkphizae5u2acjjq72g44uw2ed55gkja4 | jq
```

```json
{
  "entrypoint": "0xFAf7C4826Ee34c5f85EDE5D741CdC00Bcbf0D38F",
  "callData": "0xc67c5690000000000000000000000000e63038b36e0e588dca1459120d406914fb8154260000000000000000000000000000000000000000000000000000000000009c400000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000d18c2e28000000000000000000000000000000000000000000000000000000000000077ac34248590acc428da0ccc7615ec6f8730b8e59594e687991c562d64d74498b5b1458d3515f36d402df6b30e1cdb7e9dac332104f7696fe73507df478502206b9ec000000000000000000000000000000000000000000000000000000000000001b",
  "gasLimit": 90000,
  "feeToken": "0x0000000000000000000000000000000000000000",
  "endorser": "0xd4aAEb06A05cBCFd5c801F087Bf2561a990528ae",
  "endorserCallData": "0x",
  "endorserGasLimit": 10000000,
  "maxFeePerGas": "900000000000",
  "priorityFeePerGas": "900000000000",
  "baseFeeScalingFactor": "1",
  "baseFeeNormalizationFactor": "1",
  "hasUntrustedContext": false
}
```

## Malleability

It is important to note that the operation itself is not signed, and it is not possible to sign it, as the operation does not belong to any particular user or wallet. The responsibility of validating the operation is left to the endorser, which should be able to validate to recover the inner signature from the operation's callData.
