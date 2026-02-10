
# Lab: Upgradable Smart Contracts (Proxy Pattern)

### Objective

Deploy a contract, realize it has a "bug" or needs a new feature, and upgrade it to a new version **without changing the contract address** and **without losing data**.

---

## 1. The Theory: How it Works

When a user calls the Proxy, it doesn't know what to do. It uses a `delegatecall` to ask the Implementation contract for the logic. The key is that the code runs in the **context** (storage) of the Proxy.

---

## 2. The Smart Contracts

### Step 1: Create `ImplementationV1.sol`

This is our first version. It has a simple counter.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "@openzeppelin/contracts-upgradeable/proxy/utils/UUPSUpgradeable.sol";
import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";

contract ImplementationV1 is UUPSUpgradeable, OwnableUpgradeable {
    uint256 public count;

    function initialize() public initializer {
        __Ownable_init(msg.sender);
    }

    function increment() public {
        count += 1;
    }

    // Required by UUPS to restrict who can upgrade
    function _authorizeUpgrade(address newImplementation) internal override onlyOwner {}
}

```

### Step 2: Create `ImplementationV2.sol`

This version adds a `decrement` function and a `name` variable.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "./ImplementationV1.sol";

contract ImplementationV2 is ImplementationV1 {
    string public name;

    function setName(string memory _name) public {
        name = _name;
    }

    function decrement() public {
        count -= 1;
    }
}

```

---

## 3. Practical Steps in Remix

### Phase A: Deployment (The Initial State)

1. **Compile** `ImplementationV1`.
2. Go to the **Deploy** tab.
3. Instead of a regular deploy, use the **OpenZeppelin Proxy** tool if available, or manually deploy a `ERC1967Proxy` contract.
* *Easier Method for Students:* Use the **Remix "Deploy with Proxy"** checkbox (available in modern Remix versions).


4. Once deployed, look at the address. Let's call it `0xPROXY`.
5. Call `increment()`. Check `count`. It should be `1`.

### Phase B: The Upgrade

1. **Compile** `ImplementationV2`.
2. In the Deploy tab, select **ImplementationV2**.
3. If using the Remix Proxy UI, select **Upgrade**.
4. Paste the `0xPROXY` address into the upgrade field.
5. Confirm the transaction.

### Phase C: Verification (The "Magic Moment")

1. Interact with the **same `0xPROXY` address**.
2. **Check `count`:** It should still be `1` (Data was preserved!).
3. **Check for new functions:** You should now see `decrement()` and `setName()`.
4. Call `decrement()`. `count` should go back to `0`.

---

## 4. The "Danger Zone"

### A. The "Initializer" vs. Constructor

Explain to students that **Proxies cannot use constructors**. Why? Because constructors run during deployment and belong to the implementation's storage, not the proxy's. We use `initialize()` functions instead.

### B. Storage Collisions

This is the most dangerous part of upgradability.

* **The Rule:** You can only **add** new variables at the **end** of the contract.
* **The Demo:** Ask students what would happen if we swapped the order of `count` and `name` in V2. (Answer: The `name` string would overwrite the `count` slot, corrupting the data).

### C. Who can upgrade?

If this function isn't protected by `onlyOwner`, anyone could hijack the contract and upgrade it to a malicious version.
