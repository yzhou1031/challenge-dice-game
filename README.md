# Challenge: Dice Game
> Exploit block-level pseudo-randomness to predict outcomes and guarantee a win every roll

[![Solidity](https://img.shields.io/badge/Solidity-0.8.20-363636?logo=solidity&logoColor=white)]()
[![Foundry](https://img.shields.io/badge/Built_with-Foundry-red)]()
[![Next.js](https://img.shields.io/badge/Frontend-Next.js-black?logo=next.js&logoColor=white)]()
[![Sepolia](https://img.shields.io/badge/Network-Sepolia-8A2BE2)]()

🔗 [Live Demo](https://nextjs-rxqv6bwel-yuchenzhou1031-6631s-projects.vercel.app/) · 📋 [Speedrun Ethereum](https://speedrunethereum.com)

## What It Does

An attacker contract (`RiggedRoll`) that replicates the `DiceGame` contract's randomness formula before each roll. Because both contracts run in the same block, the predicted value is identical to the "random" outcome. `RiggedRoll` only calls `rollTheDice()` when the prediction is a winning number (0–5 out of 0–15), guaranteeing it never loses.

## Real-World Relevance

- **NFT minting fairness** — projects that use `block.prevrandao` for trait assignment are exploitable by validators or same-block attackers; this challenge demonstrates exactly how; production drops use commit-reveal schemes or Chainlink VRF to prevent it
- **PoolTogether / Chainlink VRF** — PoolTogether's no-loss lottery switched to Chainlink's Verifiable Random Function after recognizing that any on-chain randomness based on block data can be gamed by miners or same-transaction predictors
- **MEV (Miner Extractable Value)** — the ability to predict or influence block-level values is the root mechanic behind MEV; understanding this exploit is foundational to MEV-resistant protocol design

## Contract Architecture

| Contract | Role |
|---|---|
| `DiceGame.sol` | House contract; generates pseudo-random roll from `blockhash`, contract address, and public nonce; pays prize on rolls 0–5 |
| `RiggedRoll.sol` | Attacker contract; replicates the hash before calling, reverts with `NotWinningRoll` if the prediction loses, only proceeds on guaranteed wins |

## Key Concepts

- **Same-block hash replication** — `block.prevrandao` and `blockhash(block.number - 1)` are known constants within a transaction; any contract can compute the same "random" value the target will compute
- **Selective execution** — `RiggedRoll` reverts via `NotWinningRoll` on losing predictions, spending no ETH; the gas cost of a revert is the only overhead
- **Why VRF solves this** — Chainlink VRF commits to a random seed off-chain and proves it on-chain after the fact; no contract can know the value before the transaction is included

## Local Setup

```bash
yarn chain    # start local Anvil blockchain
yarn deploy   # deploy DiceGame + RiggedRoll
yarn start    # frontend at http://localhost:3000
```