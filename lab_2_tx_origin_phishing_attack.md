## Lab 2: tx.origin Phishing Attack

In this lab, we explore the difference between `msg.sender` and `tx.origin`. This vulnerability is often used in **phishing attacks** to trick a wallet owner into authorizing a transfer they didn't intend to make.

---

### 1. The Vulnerable Wallet

The owner of this wallet thinks it is secure because only the "original signer" can move funds. However, they are using the wrong global variable.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract PhishWallet {
    address public owner;

    constructor() payable {
        owner = msg.sender;
    }

    function transfer(address payable _to, uint256 _amount) public {
        // VULNERABILITY: tx.origin follows the call chain back to the original EOA.
        // If the owner calls a malicious contract, tx.origin is still the owner!
        require(tx.origin == owner, "Not the owner");

        (bool sent, ) = _to.call{value: _amount}("");
        require(sent, "Failed to send Ether");
    }

    function getBalance() public view returns (uint256) {
        return address(this).balance;
    }
}

```

---

### 2. The Attacker Contract (The Phishing Trap)

The attacker deploys this contract and lures the owner of the `PhishWallet` to interact with it (e.g., promising a "Free NFT" or "Airdrop").

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "./PhishWallet.sol";

contract MaliciousTrap {
    address payable public attacker;
    PhishWallet wallet;

    constructor(address _walletAddress) {
        wallet = PhishWallet(_walletAddress);
        attacker = payable(msg.sender); // The hacker's address
    }

    // The owner is tricked into calling this function
    function claimFreeAirdrop() public {
        // Behind the scenes, this contract calls the PhishWallet.
        // Because the Owner started the transaction, tx.origin is the Owner.
        // The PhishWallet's 'require' check will PASS.
        wallet.transfer(attacker, address(wallet).balance);
    }
}

```

---

### 3. The Fixed Wallet

The fix is simple: always use `msg.sender`. This ensures that the caller must be the owner **directly**, not a contract acting on the owner's behalf.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract FixedWallet {
    address public owner;

    constructor() payable {
        owner = msg.sender;
    }

    function transfer(address payable _to, uint256 _amount) public {
        // FIX: msg.sender is the immediate caller.
        // If the owner calls the Trap, and the Trap calls this, 
        // msg.sender will be the Trap's address, which fails the check.
        require(msg.sender == owner, "Not owner");

        (bool sent, ) = _to.call{value: _amount}("");
        require(sent, "Failed to send Ether");
    }
}

```

---

### 📋 Full Execution Steps (Remix)

1. **Deploy Wallet**: Using **Account 1** (The Victim), deploy `PhishWallet` with **10 ETH**.
2. **Deploy Trap**: Switch to **Account 2** (The Attacker). Copy the Wallet address. Paste it into the `MaliciousTrap` constructor and click "Deploy."
3. **The Phish**: Switch back to **Account 1** (The Victim). Go to the `MaliciousTrap` contract and click `claimFreeAirdrop`.
4. **The Result**:
* Check `getBalance` on the Wallet: It will be **0**.
* Check **Account 2** balance: It will have increased by 10 ETH.


5. **Verification**: Explain to the students that even though the owner "voted" for the transaction, they didn't realize which function they were actually authorizing.

---

### Security Takeaway

`tx.origin` should **never** be used for authorization. It can be useful for certain "Owner-only" checks to ensure the caller isn't a contract (e.g., `require(tx.origin == msg.sender)`), but using it to verify identity opens the door to social engineering attacks.
