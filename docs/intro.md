---
slug: /
sidebar_position: 1
---

# ERC5189 - AA Mempool using Endorsers

This is documentation for the ERC-5189 standard, it contains information about the standard, design decisions, and examples using the reference implementation. Using this documentation, you will be able to understand how to integrate any Meta-Transaction using contracts to work with the AA Mempool; additionally, you will be able to understand how to access the AA Mempool to relay transactions.

## Introduction

ERC-5189 is a standard born from the need to have a standard way to relay meta-transactions in a decentralized way. It is a standard that allows different parties to communicate with each other on what conditions a transaction can be relayed safely, with proper compensation for the relayer.

The standard attempts to achieve this goal without incurring in any additional risks for the integrators, contracts that implement this standard don't need to modify their logic to support it, as the standard is designed to be as non-intrusive as possible.

Additionally, the standard is meant to be compatible with the existing EVM networks, without requiring a hardfork or any additional changes to the network to support it.

### Design Goals

The main design goals of the standard are:

- **Decentralization**: The standard is designed to be as decentralized as possible, allowing any party to start generating and relaying meta-transactions, even if this party has no reputation or is not known by the network. Similarly, accessing the mempool is permissionless.
- **Efficiency**: The is meant to have **zero on-chain gas overhead**, allowing wallets (and other contracts) to maintain their current gas costs when relaying transactions.
- **Compatible**: The standard is designed to be compatible with most meta-transactions constructs, even including the ones that exist today. This means that contracts that implement this standard don't need to modify their logic to support it.
- **Non-intrusive**: The standard is designed to be as non-intrusive as possible, it is loosely coupled with the contracts that implement it, meaning that anyone can integrate it, even an external party that has no control over the contract.
- **Simple**: The standard attempts to achieve its goals in the simplest way possible, avoiding edge cases, reputation systems, and other complex constructs that could make the standard harder to understand and implement.

### The Problem

To better understand the standard, it is important to first understand the problem that it is trying to solve.

Wallet developers implement their wallets in a way that, a relayer, can safely relay a transaction on behalf of the user. The relayer gets compensated for the costs of relaying the transaction, even if the transaction itself where to fail.

This compensation is usually handled by the wallet itself, that separates the "verification" side of the transaction from the "execution" side of the transaction. This is performed in a myriad of ways, for different reasons, some wallets attempt to maintain the costs as low as possible, other try to provide extra features, and some may even decide to keep their logic as simple as possible, or even private.

This usually means that the wallet team also implements a "relayer service", this relayer service is responsible for relaying the transactions, and it can do it safely because it is built under the same assumptions that the wallet is built. This lets the relayer service decide which transactions are safe to relay, and which are not. These assumptions additionally allow the relayer to cheaply remove stale transactions from the mempool.

The problem arises when a wallet wants to use a relayer service that is not built by the same team that built the wallet, or when a relayer wants to relay transactions from a wallet that is not built by the same team that built the relayer service. The relayer service can't safely interpret the important details of the transaction, or it has to do it in a very expensive way.

In practice this leads to relayers that are "islands", they can only relay transactions from wallets that they understand, and developers who build these relayers have no incentive to support competing wallets, so they don't.

### ERC-5189's Solution

Some other standards, like ERC-4337, attempt to solve this problem by creating an "uniform" format for wallets, and expect both wallets and relayers to implement it. This approach works, but it is not ideal. It takes away the freedom of the wallet developers to implement their wallets in the way that they see fit, and it leads to considerable overhead for meta-transactions, as much of the logic ends up being duplicated.

Our thinking is that the problem is a problem of "communication", the developers of the wallet understand under what constraints a transaction can be safely executed, they understand their own wallets, but as it stands, they have no way to communicate this to the relayer service.

ERC-5189 solves this communication problem by introducing the concept of "endorsers". Endorsers are contracts that aren't directly linked to the wallet, but that, on demand, can validate a transaction for a particular wallet.