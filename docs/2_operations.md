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

[ipfs://bafkreicfpdvs2kgqagcqvihgsp4xjwsfrn7qefefcjwa7yttxmofzan7ly/](ipfs://bafkreicfpdvs2kgqagcqvihgsp4xjwsfrn7qefefcjwa7yttxmofzan7ly/)

By looking up the contents of this IPFS hash, you will find the JSON representation of the operation.

```
ipfs cat bafkreicfpdvs2kgqagcqvihgsp4xjwsfrn7qefefcjwa7yttxmofzan7ly | jq
```

```json
{
  "chainId": "42170",
  "data": "0x725ae723000000000000000000000000750ba8b76187092b0d1e87e28daaf484d1b5273b000000000000000000000000b70d2e30b2d2f8af8b657f5da5b305ec533f6dc2000000000000000000000000b70d2e30b2d2f8af8b657f5da5b305ec533f6dc200000000000000000000000000000000000000000000000000000000000186a0ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff0000000000000000000000000000000000000000000000000000000000029fb10000000000000000000000000000000000000000000000000000000004198bed00000000000000000000000000000000000000000000000000000000e7169db300000000000000000000000000000000000000000000000000000000000249f09dace565168424eb3b1ea70f46cd903134c1149d19338fb7f934b58d71e4cf997e0d5c89d2574899ef63559814d2830a8a9c4aa1d4e2ab96ebfbf007d4c9a9a7000000000000000000000000000000000000000000000000000000000000001c",
  "endorser": "0x274e9e4ea34cfc689dcb56d057f868f46e0ba80c",
  "endorserCallData": "0x",
  "endorserGasLimit": "10000000",
  "entrypoint": "0x61c6239d8f0d259daf7d1b0a9965a5fcd424a0cd",
  "feeNormalizationFactor": "1000000000000000000",
  "feeScalingFactor": "3877019059",
  "feeToken": "0x750ba8b76187092b0d1e87e28daaf484d1b5273b",
  "fixedGas": "0",
  "gasLimit": "150000",
  "hasUntrustedContext": false,
  "maxFeePerGas": "68783085",
  "maxPriorityFeePerGas": "171953"
}
```

## Malleability

It is important to note that the operation itself is not signed, and it is not possible to sign it, as the operation does not belong to any particular user or wallet. The responsibility of validating the operation is left to the endorser, which should be able to validate to recover the inner signature from the operation's callData.
