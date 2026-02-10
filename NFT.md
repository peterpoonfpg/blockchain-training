## Creating and Listing Your Custom NFT

This guide walks you through the professional process of storing assets decentrally, deploying a custom smart contract via Remix, and listing your NFT on OpenSea.

---

### Step 1: Prepare Your Asset and Metadata

To ensure your NFT is permanent, we use **IPFS** instead of a central server.

1. **Upload Image:** Upload your artwork to [Pinata](https://www.pinata.cloud/). Copy the **Image CID** (e.g., `QmXyz...`).
2. **Create `metadata.json`:** Create a text file with the following structure, inserting your Image CID:
```json
{
  "name": "My Custom NFT #1",
  "description": "Created using a custom smart contract in 2026.",
  "image": "ipfs://PASTE_YOUR_IMAGE_CID_HERE",
  "attributes": [
    { "trait_type": "Background", "value": "Blue" },
    { "display_type": "number", "trait_type": "Level", "value": 5, "max_value": 10 }
  ]
}

```


3. **Upload Metadata:** Upload this `metadata.json` file to Pinata. Copy the **Metadata CID**.

---

### Step 2: Deploy the Smart Contract using Remix

Using a custom contract gives you ownership over the collection identity.

1. **Open Remix:** Go to [remix.ethereum.org](https://www.google.com/search?q=https://remix.ethereum.org/).
2. **Create File:** Create `MyNFT.sol` and paste this code:
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/token/ERC721/extensions/ERC721URIStorage.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract MyCollection is ERC721URIStorage, Ownable {
    uint256 private _nextTokenId;

    constructor(address initialOwner) 
        ERC721("My2026Collection", "MNFT") 
        Ownable(initialOwner) 
    {}

    function safeMint(address to, string memory uri) public onlyOwner {
        uint256 tokenId = _nextTokenId++;
        _safeMint(to, tokenId);
        _setTokenURI(tokenId, uri);
    }
}

```


3. **Compile:** Go to the **Solidity Compiler** tab and click **Compile MyNFT.sol**.
4. **Deploy:** * Go to **Deploy & Run Transactions**.
* Set Environment to **Injected Provider - MetaMask**.
* Paste your wallet address in the `initialOwner` field.
* Click **Deploy** and sign in MetaMask.



---

### Step 3: Mint Your NFT

Once deployed, look at the "Deployed Contracts" section at the bottom of the Remix sidebar.

1. **Expand Contract:** Click the arrow next to your contract name.
2. **Run `safeMint`:**
* **to:** Your wallet address.
* **uri:** `ipfs://PASTE_YOUR_METADATA_CID_HERE`


3. **Transact:** Click the orange button and confirm in MetaMask.

---

### Step 4: List on OpenSea

1. **Visit OpenSea:** Go to [opensea.io](https://opensea.io/) and connect your wallet.
2. **Find NFT:** Go to your **Profile**. Your NFT should appear within a few minutes.
3. **Set Price:** Click on the NFT, then click **List for Sale**.
4. **Confirm:** Set your price (e.g., in ETH or MATIC) and sign the final listing transaction.

---

### Summary Table

| Step | Tool | Goal |
| --- | --- | --- |
| **Storage** | Pinata / IPFS | Make your art and data permanent. |
| **Coding** | Remix / Solidity | Create your own "rules" and collection name. |
| **Minting** | Remix / MetaMask | Put the NFT on the blockchain. |
| **Market** | OpenSea | Make the NFT visible for buyers. |
