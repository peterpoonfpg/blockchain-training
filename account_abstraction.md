# Lab: Advanced Account Abstraction (ERC-4337)

**Objective:** Transition from basic EOA (Externally Owned Account) thinking to **Programmable Wallets**. You will implement and test Social Recovery, Session Keys, and Spending Limits within a single "Smart Account" using Remix.

---

## 1. The Core Concept

In ERC-4337, the wallet is no longer just a private key—it is a **Smart Contract**. This allows us to move security and logic from the user’s brain (remembering seed phrases) into the code.

---

## 2. The Master Smart Account Contract

Create a new file in Remix named `MasterSmartAccount.sol` and paste the following code.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

/**
 * @title MasterSmartAccount
 * @dev Teaching material for ERC-4337 features: Social Recovery, Session Keys, and Spending Limits.
 */
contract MasterSmartAccount {
    address public owner;
    
    // --- 1. Social Recovery State ---
    mapping(address => bool) public isGuardian;
    address public proposedOwner;
    uint256 public constant RECOVERY_THRESHOLD = 2;
    uint256 public votesForRecovery;
    mapping(address => bool) public hasVoted;

    // --- 2. Session Key State ---
    address public sessionKey;
    uint256 public sessionExpiry;

    // --- 3. Spending Limit State ---
    uint256 public dailyLimit = 1 ether;
    uint256 public lastResetTime;
    uint256 public spentToday;

    constructor(address[] memory _guardians) payable {
        owner = msg.sender;
        for (uint i = 0; i < _guardians.length; i++) {
            isGuardian[_guardians[i]] = true;
        }
        lastResetTime = block.timestamp;
    }

    modifier onlyOwner() {
        require(msg.sender == owner, "Not the owner");
        _;
    }

    // --- FEATURE 1: SOCIAL RECOVERY ---
    function initiateRecovery(address _newOwner) external {
        require(isGuardian[msg.sender], "Not a guardian");
        proposedOwner = _newOwner;
        votesForRecovery = 1;
        hasVoted[msg.sender] = true;
    }

    function supportRecovery() external {
        require(isGuardian[msg.sender], "Not a guardian");
        require(!hasVoted[msg.sender], "Already voted");
        
        hasVoted[msg.sender] = true;
        votesForRecovery++;

        if (votesForRecovery >= RECOVERY_THRESHOLD) {
            owner = proposedOwner;
            votesForRecovery = 0; // Reset state
            proposedOwner = address(0);
        }
    }

    // --- FEATURE 2: SESSION KEYS ---
    function authorizeSession(address _key, uint256 _durationSeconds) external onlyOwner {
        sessionKey = _key;
        sessionExpiry = block.timestamp + _durationSeconds;
    }

    function executeAsSessionKey(address payable _to, uint256 _amount) external {
        require(msg.sender == sessionKey, "Not authorized session key");
        require(block.timestamp <= sessionExpiry, "Session expired");
        
        _checkAndIncreaseSpending(_amount);
        (bool success, ) = _to.call{value: _amount}("");
        require(success, "Transfer failed");
    }

    // --- FEATURE 3: SPENDING LIMITS ---
    function _checkAndIncreaseSpending(uint256 _amount) internal {
        if (block.timestamp > lastResetTime + 1 days) {
            spentToday = 0;
            lastResetTime = block.timestamp;
        }
        require(spentToday + _amount <= dailyLimit, "Exceeds daily spending limit");
        spentToday += _amount;
    }

    receive() external payable {}
}

```

---

## 3. Practical Exercises

### Exercise 1: The Spending Limit "Guardrail"

**Scenario:** You want to ensure that even if your key is leaked, a hacker cannot drain the whole wallet in one go.

1. **Deploy:** Use **Account 1** to deploy. For `_guardians`, input `["Account 2 Address", "Account 3 Address"]`.
2. **Fund:** Send **5 ETH** (or 5000 Wei for easier testing) to the contract address.
3. **Authorize:** As Account 1, call `authorizeSession` using **Account 4's** address for `3600` seconds.
4. **The Test:** Switch to **Account 4**. Try to `executeAsSessionKey` with an amount of **2 ETH**.
5. **Observation:** The transaction **reverts**. Check the console error; it should trigger the `"Exceeds daily spending limit"` message.

### Exercise 2: The "Game Session" Key

**Scenario:** You are playing a game and don't want to sign every "sword swing" transaction.

1. **Action:** As the Owner (Account 1), authorize **Account 5** for a session.
2. **Action:** Switch to **Account 5**. Call `executeAsSessionKey` for a small amount (e.g., 0.1 ETH).
3. **Observation:** The transaction **succeeds**. Account 5 acts as your temporary agent without having ownership of the wallet.

### Exercise 3: Social Recovery (Lost Key)

**Scenario:** You lost access to Account 1. You need to promote **Account 6** to be the new owner.

1. **Action:** Switch to **Account 2** (Guardian 1). Call `initiateRecovery` with **Account 6's** address.
2. **Action:** Switch to **Account 3** (Guardian 2). Call `supportRecovery`.
3. **Observation:** Click the `owner` button. It should now show **Account 6**. The "lost" account (Account 1) is no longer in control.
* **UX:** How does this model compare to a standard seed phrase recovery?
