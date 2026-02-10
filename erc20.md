## Lab 5: The ERC20 Lifecycle

**Objective:** Master the three primary actions of the ERC20 standard: **Minting**, **Transferring**, and **Delegated Spending (Approve/TransferFrom)**.

### The Token Contract

We will use the standard OpenZeppelin implementation. This is the "gold standard" for production.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract ClassroomToken is ERC20 {
    // We mint the total supply to the person who deploys the contract
    constructor(uint256 initialSupply) ERC20("ClassroomToken", "CLT") {
        _mint(msg.sender, initialSupply * 10**decimals());
    }
}

```

---

### 📋 Phase 1: Direct Transfer

This is the simplest way to move tokens. It is a peer-to-peer transaction.

1. **Deploy:** Deploy the contract with an `initialSupply` of **1000**.
2. **Check Balance:** Call `balanceOf` for **Account 1** (The Deployer).
* Result: `1000000000000000000000` (1000 tokens + 18 zeros).


3. **Transfer:** From **Account 1**, call the `transfer` function:
* `to`: **Account 2** address.
* `amount`: `500000000000000000000` (500 tokens).


4. **Verify:** Check `balanceOf` for both accounts. They should now have 500 tokens each.

---

### 📋 Phase 2: Delegated Transfer (The "DApp" Flow)

This is how Uniswap or OpenSea works. You don't "send" tokens to them; you **authorize** them to take tokens from you.

1. **Approve:** **Account 1** wants to let **Account 3** spend some of its tokens.
* Use **Account 1** in Remix.
* Call `approve`.
* `spender`: **Account 3** address.
* `amount`: `100000000000000000000` (100 tokens).


2. **Check Allowance:** Call `allowance`.
* `owner`: **Account 1**.
* `spender`: **Account 3**.
* Result: `100000000000000000000`.


3. **The "Pull" (transferFrom):** Now, switch to **Account 3**.
* Call `transferFrom`.
* `from`: **Account 1**.
* `to`: **Account 3** (itself).
* `amount`: `100000000000000000000`.


4. **Verify:** **Account 3** now has a balance of 100, and the allowance for Account 1 is now 0.

---

### 📋 Phase 3: Metadata & Decimals

Tokens are not "whole numbers" in Solidity. Students must understand the math.

| Function | Output | Explanation |
| --- | --- | --- |
| `name()` | `ClassroomToken` | The full name of your asset. |
| `symbol()` | `CLT` | The ticker symbol. |
| `decimals()` | `18` | The number of decimal places (matches ETH). |
| `totalSupply()` | `1000 * 10^18` | The total number of tokens in existence. |

