This lab will walk you through implementing an **Upgradable Smart Contract** using the **UUPS (Universal Upgradeable Proxy Standard)** pattern directly in Remix IDE.

---

## 1. The Architecture

In a proxy pattern, the state (variables) is stored in the **Proxy**, while the logic resides in the **Implementation** contract. When you upgrade, you simply point the Proxy to a new Implementation contract.

---

## 2. Lab Setup: The Implementation Contracts

We need two versions of a contract to demonstrate the upgrade.

### Version 1: `ImplementationV1.sol`

Create a new file in Remix named `ImplementationV1.sol`.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";
import "@openzeppelin/contracts-upgradeable/proxy/utils/UUPSUpgradeable.sol";
import "@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol";

contract ImplementationV1 is Initializable, UUPSUpgradeable, OwnableUpgradeable {
    uint256 public value;

    /// @custom:oz-upgrades-unsafe-allow constructor
    constructor() {
        _disableInitializers();
    }

    function initialize(uint256 _value) public initializer {
        __Ownable_init(msg.sender);
        __UUPSUpgradeable_init();
        value = _value;
    }

    function _authorizeUpgrade(address newImplementation) internal override onlyOwner {}
}

```

### Version 2: `ImplementationV2.sol`

Create another file named `ImplementationV2.sol`. This version adds an `increment` function.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "./ImplementationV1.sol";

contract ImplementationV2 is ImplementationV1 {
    // New logic for V2
    function increment() public {
        value += 1;
    }
}

```

---

## 3. Step-by-Step Deployment in Remix

### Phase A: Deploying Version 1

1. **Compile:** Go to the "Solidity Compiler" tab and compile `ImplementationV1.sol`.
2. **Deploy with Proxy:**
* Go to the "Deploy & Run Transactions" tab.
* Select `ImplementationV1` from the **Contract** dropdown.
* Look for the checkImplementation **"Deploy with Proxy"** (Remix native feature). Check it.
* Click **Deploy**.


3. **Initialization:**
* Remix will prompt you for the initialization parameters.
* Input a number (e.g., `42`) for `_value`.
* Confirm the transaction in the modal.


4. **Interact:**
* You will see two entries in "Deployed Contracts": the **ERC1967Proxy** and the **ImplementationV1**.
* **Always interact with the Proxy address.** Expand the Proxy instance; you should see the `value` is `42`.



### Phase B: Upgrading to Version 2

1. **Compile:** Compile `ImplementationV2.sol`.
2. **Perform Upgrade:**
* In the "Deploy & Run" tab, select `ImplementationV2` in the **Contract** dropdown.
* Check **"Upgrade with Proxy"**.
* In the address field that appears, ensure the address of your **already deployed Proxy** is selected.
* Click **Upgrade**.


3. **Verify:**
* Interact with the same Proxy instance again.
* You will now see the `increment` function available (you might need to "At Address" the ImplementationV2 ABI to the Proxy address if the UI doesn't refresh).
* Call `increment`, then check `value`. It should now be `43`.
