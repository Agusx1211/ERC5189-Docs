---
slug: /
sidebar_position: 1
---

# Introduction

This documentation for the ERC-5189 standard contains information about the standard, design decisions, and examples using the reference implementation. Using this documentation, you will be able to understand how to integrate any Meta-Transaction using contracts to work with the AA Mempool; additionally, you will understand how to access the AA Mempool to relay transactions.

## ERC-5189 - AA Mempool Using Endorsers

ERC-5189 is a standard born from the need to have a decentralized way to relay meta-transactions. It allows different parties to communicate information about under what conditions a transaction can be relayed safely, with proper compensation for the relayer.

The standard aims to achieve this goal without incurring any additional risks for the integrators. Contracts that implement this standard don't need to modify their logic to support it, as the standard is designed to be as non-intrusive as possible.

Additionally, the standard is meant to be compatible with existing EVM networks, without requiring a hard fork or any additional changes to the network to support it.

### Design Goals

The main design goals of the standard are:

- **Decentralization**: The standard is designed to be as decentralized as possible, allowing any party to start generating and relaying meta-transactions, even if this party has no reputation or is not known by the network. Accessing the mempool is also permissionless.
- **Efficiency**: The standard is meant to have **zero on-chain gas overhead**, allowing wallets (and other contracts) to maintain their existing gas costs.
- **Compatibility**: The standard is designed to be compatible with most meta-transactions constructs, including those that exist prior to the standard. This means that contracts that implement this standard don't need to modify their logic to support it.
- **Non-intrusiveness**: The standard is designed to be as non-intrusive as possible. It is loosely coupled with the contracts that implement it, meaning that anyone can integrate it, even an external party that has no control over the contract.
- **Simplicity**: The standard attempts to achieve its goals in the simplest way possible, avoiding edge cases, reputation systems, and other complex constructs that could make the standard harder to understand and implement.

### The Problem

To better understand the standard, it is important to first understand the problem it is attempting to solve.

Wallet developers implement their wallets in a way that allows a relayer to safely relay a transaction on behalf of the user. The relayer gets compensated for the costs of executing the transaction, even if the transaction itself were to fail.

This compensation is usually handled by the wallet contract itself, which internally separates the "verification" side of the transaction from the "execution" side. This is performed in myriad ways, for different reasons. Some wallets aim to keep costs as low as possible, others try to provide extra features, and some may decide to keep their logic as simple as possible, or even private.

This usually means that the wallet team also implements a "relayer service", responsible for relaying transactions. It can do so safely because it is built under the same assumptions as the wallet. This lets the relayer service decide which transactions are safe to relay. These assumptions also allow the relayer to cheaply remove stale transactions from the mempool.

The problem arises when a wallet wants to use a relayer service that is not built by the same team that built the contracts, or when a relayer wants to relay transactions from a wallet not built by the same team. The relayer service can't safely interpret the important details of the transaction, or it has to do so in a very expensive way.

In practice, this leads to relayers that are "islands", only able to relay transactions from wallets they understand. Since developers who build these relayers have no incentive to support competing wallets, they don't do it.

### ERC-5189's Solution

Some other standards, like ERC-4337, attempt to solve this problem by creating a "uniform" format for wallets and expect both wallets and relayers to implement it. This approach works, but it is not ideal. It takes away the freedom of wallet developers to implement their wallets in the way they see fit, and it leads to considerable overhead for meta-transactions, as much of the logic ends up being duplicated and forced to be on-chain.

Our thinking is that the problem is a problem of "communication": the developers of the wallet understand under what constraints a transaction can be safely executed; they understand their own wallets, but as it stands, they have no way to communicate this to the relayer service.

ERC-5189 solves this communication problem by introducing the concept of "endorsers". Endorsers are contracts that aren't directly linked to the wallet but can, on demand, validate a transaction for a particular wallet.

#### Endorsers as communication channels

Imagine this abstract scenario: You are a "relayer" and you receive a transaction from a wallet, but you have no idea how this wallet works, so you don't know if the transaction may be safely relayed.

You can attempt to decode and simulate this unknown transaction, and that will get you far enough when you have a handful of transactions. But simulating a full transaction is expensive, and when you have to do it for hundreds of transactions, it becomes unfeasible.

You could try to take shortcuts, like recording any storage accesses that the transaction does, and only re-simulating the transaction only when some of those storage values change. But that will lead to a lot of false positives, as the transaction may access storage that is not relevant to the transaction's execution result.

Another option could be to ask whoever built the transaction to provide you with a sort of "access list" of any storage slots that may be relevant to the transaction's execution. But what happens if this list is incorrect? You can only trust it on an optimistic basis, and bad actors can negate them entirely.

That's where **Endorsers** come in: they are contracts that dynamically generate these "access lists" (dependencies) for any operation, and bundlers can use this information to take shortcuts when validating (and re-validating) transactions, since transactions from a well-behaved endorser must remain valid as long as the dependencies remain the same.

However, **Endorsers** can be penalized. Attempting to contaminate the mempool with operations from a malicious endorser will only have a temporary effect, as the mempool peers will quickly detect that the endorser is "lying" about the dependencies of the operations, and will stop accepting operations from it.

## Summary

We believe that the biggest challenge for implementing a native AA mempool is to solve the communication problem between wallets and relayers, and we believe that the best way to solve this problem is by introducing the concept of "endorsers", that can dynamically generate "access lists" for any operation, and that can be penalized if they attempt to contaminate the mempool with operations that are not valid.
