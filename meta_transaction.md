Excellent! Since the script is working, here is the polished, final version of the lab guide in Markdown format. You can provide this directly to your students as their practical handout.

---

# Lab: Gasless Meta-Transactions with EIP-712

**Objective:** Understand how users can interact with a smart contract without holding ETH by signing a "Typed Data" message and allowing a Relayer to pay the gas on the Sepolia Testnet.

---

## 1. The Smart Contract

We use the **OpenZeppelin EIP712** library. This handles the complex prefixing () and domain separator logic required to prevent signatures from being reused on other chains or contracts.

### Step 1: Create the Contract

1. Open [Remix IDE](https://www.google.com/search?q=https://remix.ethereum.org/).
2. Create a file named `SepoliaMetaTx.sol` and paste the code below:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

import "@openzeppelin/contracts/utils/cryptography/ECDSA.sol";
import "@openzeppelin/contracts/utils/cryptography/EIP712.sol";

contract SepoliaMetaTx is EIP712 {
    using ECDSA for bytes32;

    string public message;
    mapping(address => uint256) public nonces;

    // TypeHash for the UpdateMessage struct
    bytes32 private constant MESSAGE_TYPEHASH = keccak256("UpdateMessage(address user,string content,uint256 nonce)");

    constructor() EIP712("SepoliaMetaTx", "1") {}

    function executeMetaTx(
        address user,
        string memory content,
        uint256 nonce,
        bytes memory signature
    ) public {
        // 1. Replay Protection
        require(nonces[user] == nonce, "Invalid nonce");
        
        // 2. Hash the data according to EIP-712
        bytes32 structHash = keccak256(abi.encode(
            MESSAGE_TYPEHASH, 
            user, 
            keccak256(bytes(content)), 
            nonce
        ));
        
        bytes32 digest = _hashTypedDataV4(structHash);

        // 3. Recover the signer
        address signer = digest.recover(signature);
        require(signer == user, "Signature verification failed");

        // 4. Update state and increment nonce
        nonces[user]++;
        message = content;
    }
}

```

### Step 2: Deployment

1. Set **Environment** to `Injected Provider - MetaMask`.
2. Ensure you are on the **Sepolia** Network.
3. **Deploy** the contract and copy the **Contract Address**.

---

## 2. Generating the Signature (The "User" Step)

The user signs the data off-chain using MetaMask. This costs **zero gas**.

1. In your browser (where Remix is open), press `F12` to open the **Console**.
2. Paste the following script. **Update the `contractAddress` variable** with your deployed address.

```javascript
async function generateMetaTx() {
  const accounts = await ethereum.request({ method: 'eth_requestAccounts' });
  const user = accounts[0]; 
  const contractAddress = "PASTE_YOUR_CONTRACT_ADDRESS_HERE";

  const domain = {
    name: 'SepoliaMetaTx',
    version: '1',
    chainId: 11155111,
    verifyingContract: contractAddress
  };

  const types = {
    UpdateMessage: [
      { name: 'user', type: 'address' },
      { name: 'content', type: 'string' },
      { name: 'nonce', type: 'uint256' }
    ]
  };

  const message = {
    user: user,
    content: 'Gasless Transaction on Sepolia!',
    nonce: 0 
  };

  const signature = await ethereum.request({
    method: 'eth_signTypedData_v4',
    params: [user, JSON.stringify({ 
        types: { EIP712Domain: [
            { name: 'name', type: 'string' }, { name: 'version', type: 'string' },
            { name: 'chainId', type: 'uint256' }, { name: 'verifyingContract', type: 'address' }
        ], ...types }, 
        domain, 
        primaryType: 'UpdateMessage', 
        message 
    })],
  });

  console.log("--- SUCCESS! COPY THESE TO REMIX ---");
  console.log("user:", user);
  console.log("content:", message.content);
  console.log("nonce:", message.nonce);
  console.log("signature:", signature);
}
generateMetaTx();

```

---

## 3. Executing the Meta-Transaction (The "Relayer" Step)

1. **Switch Accounts:** In MetaMask, change to a **different account** (the Relayer) that has Sepolia ETH.
2. In Remix, go to the `executeMetaTx` function and paste the values exactly as they appeared in the console:
* **user:** (The signer's address)
* **content:** `Gasless Transaction on Sepolia!`
* **nonce:** `0`
* **signature:** (The long `0x...` string)


3. Click **transact**.

---

## 4. Key Security Concepts for Students

* **Signature Verification:** The `recover` function uses elliptic curve math to find the public key that signed the hash. If the hash (data) is changed by even one character, the recovered address will be wrong.
* **Nonces:** Why did we use `nonces[user]++`? Without this, a malicious Relayer could "replay" your signature to update the message multiple times without your permission.
* **Domain Separator:** The `chainId` in the script ensures that a signature meant for Sepolia cannot be used on Ethereum Mainnet (where the contract might also exist).
