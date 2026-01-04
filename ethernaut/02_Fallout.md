# Ethernaut — Fallout (Level) — What the bug was + how to solve it

## What the level is about
This level demonstrates how **a misnamed constructor** can completely break contract initialization and allow anyone to claim ownership after deployment.

In Solidity versions **before 0.7.0**, constructors are defined by a function having the **same name as the contract**.  
If the name is even slightly different, it is treated as a **normal public function** instead of a constructor.

In this contract, the intended constructor is:

function Fal1out() public payable

But the contract name is `Fallout` (with a lowercase L), while the function is `Fal1out` (with the digit `1` instead of `l`).  
Therefore:
- `Fal1out` is NOT a constructor.
- It is a normal `public payable` function callable by anyone at any time.
- It sets `owner = msg.sender`.

So the bug is: **anyone can call `Fal1out()` and become the owner**.

---

## The real vulnerability (the flaw)

### Misnamed constructor
In Solidity 0.6.0:
- A constructor must be either:
  - Named exactly like the contract (old style), or
  - Declared with the `constructor` keyword.

Here:
- Contract name: `Fallout`
- Function name: `Fal1out` (with `1` instead of `l`)

They don’t match → not a constructor → publicly callable.

When you call `Fal1out()`:
- `owner = msg.sender`
- `allocations[msg.sender] = msg.value`

So you instantly become the owner.

---

## Commands to solve the level (Ethernaut browser console)

```
/**
 * 1) Call the wrongly named "constructor" function.
 *    This sets you as owner.
 */
await contract.Fal1out({ value: "0" })

/**
 * 2) Verify ownership
 */
await contract.owner()

/**
 * 3) (Optional) Drain the contract as the new owner
 */
await contract.collectAllocations()
```

---

## Summary

- The contract was never properly initialized.
- The "constructor" is just a public function because its name is wrong.
- Anyone can call it and become owner.
- Lesson: Always use the `constructor` keyword and never rely on name matching.