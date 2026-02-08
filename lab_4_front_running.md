## Lab 4: Front-Running (Blind Auction)

In this final lab, we address **Front-Running**, a network-level attack where an observer (or a bot) sees your pending transaction in the **Mempool** and "jumps the line" by paying a higher gas fee to have their transaction processed first.

---

### 1. The Vulnerable Auction

This contract is "transparent." Anyone can see your bid value while it is waiting to be mined and simply submit a higher bid to beat you.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract SimpleAuction {
    address public highestBidder;
    uint256 public highestBid;

    function bid() external payable {
        // VULNERABILITY: An attacker sees your 'msg.value' in the mempool
        // and sends a slightly higher value with more gas.
        require(msg.value > highestBid, "Bid too low");

        highestBidder = msg.sender;
        highestBid = msg.value;
    }
}

```

---

### 2. The Fix: Commit-Reveal Scheme

To prevent front-running, we split the process into two phases.

1. **Commit Phase:** You submit a secret hash of your bid. No one knows your price.
2. **Reveal Phase:** You reveal your price and secret. The contract verifies it against the hash.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

contract BlindAuction {
    mapping(address => bytes32) public commitments;
    address public highestBidder;
    uint256 public highestBid;

    // STEP 1: Submit a hash of (value + secret)
    // Front-runners see the hash, but they can't see the price!
    function commitBid(bytes32 _hashedBid) external {
        commitments[msg.sender] = _hashedBid;
    }

    // STEP 2: Prove what your bid was
    function reveal(uint256 _value, string memory _secret) external {
        // Verify the data matches the previously submitted hash
        bytes32 check = keccak256(abi.encodePacked(_value, _secret));
        require(check == commitments[msg.sender], "Reveal does not match commit!");

        if (_value > highestBid) {
            highestBid = _value;
            highestBidder = msg.sender;
        }
    }
}

```

---

### 📋 Full Execution Steps (Remix)

#### Step A: Generate a Hash

Since we aren't using a frontend, we will use a small helper function or the Remix console to generate our "Secret Hash."

1. In the Remix Console, type:
`ethers.utils.solidityKeccak256(["uint256", "string"], [100, "mysecret"])`
2. Copy the resulting hash (e.g., `0xabc...`).

#### Step B: The Commit Phase

1. **Deploy** `BlindAuction`.
2. **Account 1**: Paste your hash into `commitBid` and click transact.
3. **The Lesson**: Note that if you look at this transaction on a block explorer, the "value" of the bid is hidden behind that hash. An attacker cannot "outbid" you because they don't know if you bid 1 or 1,000.

#### Step C: The Reveal Phase

1. **Account 1**: Call `reveal` with the number `100` and the string `"mysecret"`.
2. **Verify**: Check `highestBid`. It should now be `100`.

---

### Security Takeaway

On a public blockchain, **privacy does not exist by default**. If the success of your logic depends on a secret (like a bid, a move in Rock-Paper-Scissors, or a password), you must use **Commit-Reveal** or **Zero-Knowledge Proofs** to hide that data from the mempool.

---

### 🎓 Course Conclusion

You have now covered the "Big Four" of Smart Contract Security:

1. **Reentrancy**: Fix with CEI pattern.
2. **Authorization**: Fix by avoiding `tx.origin`.
3. **Arithmetic**: Fix by using Solidity 0.8+.
4. **Front-Running**: Fix with Commit-Reveal.
