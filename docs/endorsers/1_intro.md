---
slug: /endorsers
sidebar_position: 1
---

# Introduction

Endorsers are contracts with the ability to validate operations. They are the core of the ERC-5189 standard. They can take any operation, and determine two things:

- Is this operation ready to be executed?
- What are the dependencies of this operation?

In this section, we will cover the basics of endorsers, how they work and how to write one.

## Core Concepts

### Wallet and Endorser Relationship

There is no strict tie between a wallet (or any other entrypoint) and the endorser that it uses. Any operation can technically be pointing to any endorser. This is by design, as it allows the wallet developer to iterate on the endorser without needing to update the wallet.

This also means that the endorser must take care to validate the whole operation, as any operation may be pointing to it.

### Endorser Registry

The endorser registry is a singleton contract that allows anyone to register an endorser. It requires burning a small amount of native tokens, to prevent spam.

See the [Endorser Registry](./endorser-registry.md) section for more information, and for alternative ways to register an endorser.

### Endorser Interface

The endorser interface *MUST* be implemented by any contract that wants to be an endorser. This interface is simple, and only requires one method: `isOperationReady`.

```solidity
interface Endorser {
  struct GlobalDependency {
    bool baseFee;
    bool blobBaseFee;
    bool chainId;
    bool coinBase;
    bool difficulty;
    bool gasLimit;
    bool number;
    bool timestamp;
    bool txOrigin;
    bool txGasPrice;
    uint256 maxBlockNumber;
    uint256 maxBlockTimestamp;
  }

  struct Constraint {
    bytes32 slot;
    bytes32 minValue;
    bytes32 maxValue;
  }

  struct Dependency {
    address addr;
    bool balance;
    bool code;
    bool nonce;
    bool allSlots;
    bytes32[] slots;
    Constraint[] constraints;
  }

  struct Operation {
    address entrypoint;
    bytes callData;
    uint256 fixedGas;
    uint256 gasLimit;
    address endorser;
    bytes endorserCallData;
    uint256 endorserGasLimit;
    uint256 maxFeePerGas;
    uint256 priorityFeePerGas;
    address feeToken;
    uint256 feeScalingFactor;
    uint256 feeNormalizationFactor;
    bool hasUntrustedContext;
  }

  function isOperationReady(
    Operation calldata _operation,
  ) external returns (
    bool readiness,
    GlobalDependency memory globalDependency,
    Dependency[] memory dependencies
  );

  struct Replacement {
    address oldAddr;
    address newAddr;
    SlotReplacement[] slots;
  }

  struct SlotReplacement {
    bytes32 slot;
    bytes32 value;
  }

  function simulationSettings() external view returns (
    Replacement[] memory replacements
  );
}
```

> **Note**: The endorser interface also defines a `simulationSettings` method, which is optional.
