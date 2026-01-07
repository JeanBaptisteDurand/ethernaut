# Ethernaut — Token (Level) — What the bug was + how to solve it

## What the level is about
This level teaches a **classic integer underflow vulnerability** in Solidity versions **before 0.8.0**.

You start with **20 tokens**, and the goal is to end up with **more tokens than you started with**, ideally a very large amount.

The contract looks simple and safe at first glance, but it hides a critical arithmetic flaw.

---

## The vulnerable contract

```
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

contract Token {
    mapping(address => uint256) balances;
    uint256 public totalSupply;

    constructor(uint256 _initialSupply) public {
        balances[msg.sender] = totalSupply = _initialSupply;
    }

    function transfer(address _to, uint256 _value) public returns (bool) {
        require(balances[msg.sender] - _value >= 0);
        balances[msg.sender] -= _value;
        balances[_to] += _value;
        return true;
    }

    function balanceOf(address _owner) public view returns (uint256 balance) {
        return balances[_owner];
    }
}
```

---

## The real vulnerability (the flaw)

The bug is in this line:

```
require(balances[msg.sender] - _value >= 0);
```

### Why this check is useless
- `balances[msg.sender]` is a `uint256`
- `_value` is also a `uint256`
- **Unsigned integers can never be negative**

So when `_value` is **greater than your balance**, this happens:

- `balances[msg.sender] - _value` **underflows**
- Instead of reverting, it wraps around to a **huge number**
- The `require` condition still passes because the result is ≥ 0

👉 This is how integer arithmetic worked **before Solidity 0.8.0**.

---

## What is an odometer? (hint explained)

An odometer rolls over when it reaches its maximum value.

Unsigned integers behave the same way:

- `0 - 1` becomes  
  `2^256 - 1`

So instead of losing tokens, you suddenly gain an **astronomical amount**.

---

## How the exploit works

You start with:
- Balance = `20`

You call:

```
transfer(anyAddress, 21)
```

### What happens internally

1. `balances[msg.sender] - 21`
   - `20 - 21` underflows
   - Result = `2^256 - 1`
2. `require(...)` passes ✅
3. Balance update:
   - Your balance becomes a **huge number**
   - The recipient also gets tokens

🎉 Level beaten.

---

## How to exploit it (step-by-step)

1) Open the **Token** level in Ethernaut  
2) Check your balance:
```
balanceOf(player)
```
→ returns `20`

3) Call `transfer()` with **more than your balance**:
```
transfer(player, 21)
```
(or any address, even yourself)

4) Check your balance again:
```
balanceOf(player)
```
→ You now have a **massive amount of tokens**

---

## Why this works only on old Solidity versions

- Solidity `< 0.8.0`  
  ❌ No automatic overflow/underflow checks
- Solidity `>= 0.8.0`  
  ✅ Arithmetic reverts automatically on overflow/underflow

This contract uses:
```
pragma solidity ^0.6.0;
```
So it is vulnerable.

---

## Summary

- The contract relies on a **broken arithmetic check**
- Unsigned integers **wrap around** on underflow
- Sending more tokens than you own gives you **infinite tokens**
- This is a textbook example of why SafeMath existed

---

## Bonus: the proper fix

### Option 1 — correct require check

```
require(balances[msg.sender] >= _value, "Not enough balance");
```

### Option 2 — use Solidity ≥ 0.8.0 (recommended)

```
pragma solidity ^0.8.0;
```

Overflow and underflow will revert automatically.

---

## Lesson learned

Never trust arithmetic in old Solidity versions.

**Always assume integers can wrap unless explicitly protected.**
