Here are four structured lab guides in Markdown format, ready to be handed out to your students. Each one includes the objective, the code, and the step-by-step instructions for the **Remix IDE**.

---

# Lab 1: Reentrancy Attack

**Objective:** Learn how "Checks-Effects-Interactions" prevents an attacker from recursively draining a contract.

### 1. The Vulnerable Contract

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

        // VULNERABILITY: External call happens BEFORE balance is updated
        (bool sent, ) = msg.sender.call{value: bal}("");
        require(sent, "Failed to send");

        balances[msg.sender] = 0;
    }

    function getBalance() public view returns (uint256) {
        return address(this).balance;
    }
}

```

### Which lab would you like me to expand on with a grading rubric or a quiz for your students?
