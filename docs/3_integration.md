---
slug: /integration
sidebar_position: 3
---

# Integrations

There are three main ways to integrate with ERC-5189, depending on the role that you want to play in the network.

### Endorser - Wallet and Smart Contract Integration

If you are a wallet developer, you will want to write an endorser contract for your wallet or system. This will allow you to wrap your transactions into an `operation` that can be relayed to the ERC-5189 mempool. This will provide your users with a decentralized way to relay their transaction, so you can focus on building the wallet and not on building a relayer service.

Writing an endorser is a simple process, and mistakes are very forgiving. Endorsers don't need to have any special permissions, so they can be written by anyone, and can be deployed without expensive audits.

### Bundler - Access to the Mempool

Anyone can access the mempool and start relaying transactions. If you are a MEV searcher, a sequencer or a block producer of any kind, you can access the mempool to fetch profitable operations and execute them. This will allow you to earn fees by executing transactions on behalf of others.

### Node - Participate in the Network

You can run a ERC-5189 node to participate in the network. This will allow you to relay operations without intermediaries. Additionally, you can help propagate and archive operations, this helps the network to be more resilient and censorship-resistant.
