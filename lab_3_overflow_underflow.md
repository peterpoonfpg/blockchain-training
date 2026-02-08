## Lab 3: Arithmetic Overflow & Underflow

This lab explores how Solidity handles mathematical boundaries. In older versions of Solidity (pre-0.8.0), numbers would "wrap around" if they exceeded their maximum or minimum capacity. Modern Solidity has built-in protection, but it is vital to understand the logic for security audits and gas optimization.

---

### 1. The Experimental Contract

We use `uint8` for this demo because its range is small (0 to 255), making it very easy to "hit the ceiling" or "hit the floor." We use the `unchecked` keyword to tell the compiler to ignore safety checks, simulating older, vulnerable code.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract MathVault {
    // uint8 can only hold numbers from 0 to 255
    mapping(address => uint8) public balances;

    function deposit(uint8 _amount) public {
        // VULNERABILITY: Overflow
        // If balance is 250 and you add 10, it wraps around to 4.
        unchecked {
            balances[msg.sender] += _amount;
        }
    }

    function withdraw(uint8 _amount) public {
        // VULNERABILITY: Underflow
        // If balance is 0 and you subtract 1, it wraps around to 255.
        unchecked {
            balances[msg.sender] -= _amount;
        }
    }

    function getBalance(address _user) public view returns (uint8) {
        return balances[_user];
    }
}

```

---

### 2. The Attack (Logic)

In this scenario, an attacker doesn't need a "malicious contract." They simply exploit the math:

1. **Underflow:** An attacker with 0 balance calls `withdraw(1)`. The contract subtracts 1 from 0. Instead of failing, the balance "rolls back" to the highest possible number (**255**).
2. **Overflow:** If a user's balance is at the limit (255) and they receive more, their balance resets to 0, potentially "deleting" their funds.

---

### 3. The Fix: Native 0.8.x Protection

In modern Solidity, the `unchecked` block is removed. The compiler automatically adds bytecode that checks for these conditions and **reverts** the transaction if math fails.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract SafeMathVault {
    mapping(address => uint8) public balances;

    // FIXED: No 'unchecked' block. 
    // If (balance + amount) > 255, the transaction REVERTS.
    function deposit(uint8 _amount) public {
        balances[msg.sender] += _amount;
    }

    // FIXED: No 'unchecked' block.
    // If (balance - amount) < 0, the transaction REVERTS.
    function withdraw(uint8 _amount) public {
        balances[msg.sender] -= _amount;
    }
}

```

---

### 📋 Full Execution Steps (Remix)

1. **Deploy `MathVault**`: Deploy the vulnerable version.
2. **The Underflow**:
* Ensure your balance is 0 (check `getBalance`).
* Call `withdraw` with the value `1`.
* Check `getBalance` again. It will now show **255**.


3. **The Overflow**:
* While your balance is 255, call `deposit` with the value `1`.
* Check `getBalance`. It will now show **0**.


4. **The Fix**:
* Deploy `SafeMathVault`.
* Try to `withdraw(1)` while at 0 balance.
* Observe the **Remix Console**: The transaction will fail with a "Panic" error, and the balance remains 0.



---

### Security Takeaway

Always use the latest Solidity compiler. Only use `unchecked` blocks if you have manually verified the math is safe and you need to save a small amount of gas (e.g., incrementing a loop counter that will never reach ).
