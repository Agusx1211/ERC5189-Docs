---
slug: /endorsers/building_an_endorser
sidebar_position: 3
---

# Building an Endorser

In this section we will cover the basics of building a minimal endorser. We will cover what tools the endorser has at its disposal, and what it needs to do to validate an operation, along with some best practices.

## Demo wallet

Endorsers don't exist in a vacuum. They are always used by a wallet or another entrypoint. They rely on using the same assumptions and the same data as the wallet. To start building an endorser, we will use a simple wallet as a reference.

This wallet is not going to gain any awards for security, features or efficiency. It is just a simple wallet that we can use to test our endorser.

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;


contract DemoWallet {
  address public owner;

  mapping(uint256 => bool) public usedNonces;

  constructor(address _owner) {
    owner = _owner;
  }

  function send(
    address payable _to, uint256 _gasLimit, uint256 _amount, uint256 _fee, uint256 _nonce,
    bytes32 _r, bytes32 _s, uint8 _v
  ) external {
    if (gasleft() < _gasLimit) {
      revert("insufficient gas");
    }

    bytes32 txhash = keccak256(abi.encode(_to, _gasLimit, _amount, _fee, _nonce));
    require(ecrecover(txhash, _v, _r, _s) == owner, "invalid signature");

    if (usedNonces[_nonce]) {
      revert("used nonce");
    }

    usedNonces[_nonce] = true;

    _to.call{value: _amount}("");

    (bool ok2, ) = payable(tx.origin).call{value: _fee}("");
    require(ok2);
  }
}
```

This simple wallet can only do two things: send funds and pay fees. It has a simple nonce mechanism to prevent replay attacks. It also has a simple signature verification mechanism.

A few corners have been cut to keep the code simple. For example, the wallet always pays a fixed fee, it doesn't provide cross-chain replay protection and it doesn't do anything else apart from sending native tokens.

### Abstract endorser

In this scenario, the endorser will need to verify the following things:

- The entrypoint is a valid wallet.
- The signature is valid.
- The nonce is not used.
- The gas limit is sufficient.
- The wallet has enough funds to pay for everything.

Additionally the endorser will need to return a list of dependencies that may invalidate the operation:

- The wallet's balance.
- The used nonce.

Notice how the `owner` is not a dependency, even though it is accessed by the wallet to verify the signature. In this case, the endorser knows that the wallet does not allow for the `owner` to change, so it can safely not include it as a dependency.

### Endorser implementation

