---
slug: /endorsers/quickstart
sidebar_position: 2
---

# Quickstart

Endorsers are contract like any other, they can be written in any language that compiles to EVM bytecode.

However, we recommend using Solidity, alongside the `LibDc` library. This library provides a set of tools that make it easier to write the list of constraints and dependencies that the endorser needs to validate an operation.

Install the [LibDc](https://github.com/0xsequence/erc5189-libs) library

```sh
forge install https://github.com/0xsequence/erc5189-libs
```

The [ERC5189-Libs](https://github.com/0xsequence/erc5189-libs) toolset includes a definition of the `Endorser` interface, as shown in the previous section. It can be accessed by importing the `IEndorser` interface from the `LibDc` library.

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

import { IEndorser } from "erc5189-libs/interfaces/IEndorser.sol";


contract Endorser is IEndorser {
  function isOperationReady(
    Operation calldata operation
  ) external returns (
    bool readiness,
    GlobalDependency memory globalDependency,
    Dependency[] memory dependencies
  ) {

  }

  function simulationSettings() external returns (
    Replacement[] memory replacements
  ) {

  }
}
```

## Building dependencies

The `isOperationReady` returns a list of dependencies and constraints, building the list of dependencies manually can be a tedious task, more so when the endorser is complex and has many dependencies. For this reason, the [ERC5189-Libs](https://github.com/0xsequence/erc5189-libs) toolset includes the `LibDc` library.

The `LibDc` library revolves around the "DependencyCarrier" or `Dc`, which is a builder helper that makes it easier to generate `GlobalDependency` and `Dependency` lists.

Using the `Dc` builder is as simple as creating a new instance of it, the `operation` can optionally be passed to it. Then, the `Dc` instance can be built into a `GlobalDependency` or `Dependency` list.

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

import { IEndorser } from "erc5189-libs/interfaces/IEndorser.sol";
import { Dc, LibDc } from "erc5189-libs/LibDc.sol";


contract Endorser is IEndorser {
  using LibDc for *;

  function isOperationReady(
    Operation calldata operation
  ) external returns (
    bool readiness,
    GlobalDependency memory,
    Dependency[] memory
  ) {
    Dc memory dc = LibDc.create(operation);

    // ... add dependencies and constraints here

    return dc.build();
  }
}
```

### Adding Dependencies

Dependencies are pointers to values that *MAY* invalidate the operation. Any condition that may invalidate the operation should be added as a dependency.

However, not everything that the wallet accesses must be added as a dependency. For example, if the wallet interacts with Uniswap, the endorser does not need to add Uniswap as a dependency, as long as the operation still pays the fee even if the Uniswap transaction fails.

```solidity

```