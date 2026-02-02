---
sidebar_position: 2
title: Create token
---

# How to Create Own Token

## Overview

Create a custom **SPL Token** using the **Token-2022 Program** programmatically with **Solana Web3.js**. Token-2022 supports advanced extensions like **transfer fees**, **metadata pointers**, and **confidential transfers** for enhanced functionality. This guide focuses on creating a basic mint with a **metadata pointer** extension for token name/symbol/description.

- **Program ID**: `TokenzQdBNbLqP5VEhdkAS6EPFLC1PHnBqCXEpPxuEb`
- **Network**: Use Pyra Chain RPC
- **Requirements**: Wallet with $PYRA for rent exemption

**Note**: This uses **@solana/spl-token** for Token-2022 extensions

---

## Installation

Install dependencies:

```
npm install @solana/web3.js @solana/spl-token @solana/spl-token-metadata
```

---

## Step-by-Step Code Example

### 1. Setup Connection & Keys
Connect to Pyra Chain and generate payer keypair.

```javascript
import {
  Connection,
  Keypair,
  PublicKey,
  SystemProgram,
  Transaction,
  LAMPORTS_PER_SOL
} from '@solana/web3.js';
import {
  ExtensionType,
  createInitializeMintInstruction,
  createAccount,
  getMintLen,
  TOKEN_2022_PROGRAM_ID,
  createInitializeMetadataPointerInstruction,
  getMintLenFromRent
} from '@solana/spl-token';
import {
  createInitializeInstruction,
  pack,
  TokenMetadata
} from '@solana/spl-token-metadata';

// Connect to Pyra Chain RPC
const connection = new Connection('https://rpc.pyrachain.io', 'confirmed');

// Payer (wallet) - Replace with your keypair
const payer = Keypair.generate();  // Or load from secret key
```

### 2. Define Token Metadata
Set token details for metadata extension.

```javascript
// Token metadata
const tokenMetadata = {
  name: 'My Custom Token',
  symbol: 'MCT',
  uri: 'https://example.com/metadata.json',  // JSON with image/description
  sellerFeeBasisPoints: 0,  // Royalty %
  creators: null,  // Optional creators array
  collection: null,  // Optional collection
  uses: null  // Optional uses config
} as TokenMetadata;

// Pack metadata for on-chain storage
const metadataDef = {
  updateAuthority: payer.publicKey,
  mint: new PublicKey('')  // Will be set later
};
const metadataLen = pack(metadataDef, [tokenMetadata]);
```

### 3. Calculate Space & Rent
Compute required space for mint with extensions.

```javascript
// Extensions: Metadata Pointer
const extensions = [ExtensionType.MetadataPointer];

// Calculate mint length
const mintLen = getMintLen([ExtensionType.MetadataPointer]);

// Get rent exemption lamports
const mintLamports = await getMintLenFromRent(connection, TOKEN_2022_PROGRAM_ID, extensions);
console.log(`Rent for mint: ${mintLamports} lamports`);
```

### 4. Create Mint Account
Generate mint keypair and fund it.

```javascript
// Generate mint keypair
const mint = Keypair.generate();

// Create mint account
const createMintAccountInstruction = SystemProgram.createAccount({
  fromPubkey: payer.publicKey,
  newAccountPubkey: mint.publicKey,
  space: mintLen,
  lamports: mintLamports,
  programId: TOKEN_2022_PROGRAM_ID
});
```

### 5. Initialize Mint with Extensions
Add instructions for mint initialization and metadata pointer.

```javascript
// Initialize mint (decimals: 9, mint authority: payer)
const decimals = 9;
const initializeMintInstruction = createInitializeMintInstruction(
  mint.publicKey,
  decimals,
  payer.publicKey,  // Mint authority
  payer.publicKey,  // Freeze authority (optional, set null to disable)
  TOKEN_2022_PROGRAM_ID
);

// Initialize metadata pointer
const initializeMetadataPointerInstruction = createInitializeMetadataPointerInstruction(
  mint.publicKey,
  payer.publicKey,  // Authority
  metadataDef.mint,  // Metadata address (derive from mint)
  TOKEN_2022_PROGRAM_ID
);
```

### 6. Optional: Add Transfer Fee Extension
For Token-2022 transfer fees (e.g., 1% fee).

```javascript
import { createInitializeTransferFeeConfigInstruction } from '@solana/spl-token';

// Transfer fee config: 50 basis points (0.5%), max 1 $PYRA per transfer
const feeBasisPoints = 50;
const maxFee = 1 * 10 ** decimals;
const transferFeeConfigAuthority = payer.publicKey;
const withdrawWithheldAuthority = payer.publicKey;

const initializeTransferFeeInstruction = createInitializeTransferFeeConfigInstruction(
  mint.publicKey,
  transferFeeConfigAuthority,
  withdrawWithheldAuthority,
  feeBasisPoints,
  maxFee,
  TOKEN_2022_PROGRAM_ID
);

// Add to extensions array above
const extensionsWithFee = [ExtensionType.TransferFeeConfig, ExtensionType.MetadataPointer];
```

### 7. Build & Send Transaction
Combine instructions and send.

```javascript
// Build transaction
const transaction = new Transaction().add(
  createMintAccountInstruction,
  initializeMintInstruction,
  initializeMetadataPointerInstruction,  // + initializeTransferFeeInstruction if using fees
  createInitializeInstruction(  // Initialize metadata account
    metadataAddress,  // Derive: findProgramAddressSync([mint.publicKey.toBuffer()], METADATA_PROGRAM_ID)
    payer.publicKey,
    mint.publicKey,
    TOKEN_METADATA_PROGRAM_ID
  )
);

// Sign and send
const signature = await connection.sendTransaction(transaction, [payer, mint]);
await connection.confirmTransaction(signature, 'confirmed');

console.log(`Mint created! Address: ${mint.publicKey.toBase58()}`);
console.log(`Transaction: ${signature}`);
```

### 8. Mint Initial Supply
Mint tokens to an associated token account.

```javascript
import { createMintToInstruction, getOrCreateAssociatedTokenAccount } from '@solana/spl-token';

// Get or create ATA for payer
const tokenAccount = await getOrCreateAssociatedTokenAccount(
  connection,
  payer,
  mint.publicKey,
  payer.publicKey,
  undefined,
  undefined,
  undefined,
  undefined,
  TOKEN_2022_PROGRAM_ID
);

// Mint 1000 tokens
const initialSupply = 1000 * 10 ** decimals;
const mintToInstruction = createMintToInstruction(
  mint.publicKey,
  tokenAccount.address,
  payer.publicKey,  // Mint authority
  initialSupply,
  [],
  TOKEN_2022_PROGRAM_ID
);

const mintTx = new Transaction().add(mintToInstruction);
const mintSig = await connection.sendTransaction(mintTx, [payer]);
await connection.confirmTransaction(mintSig);

console.log(`Minted ${initialSupply / 10 ** decimals} tokens to ${tokenAccount.address.toBase58()}`);
```
---

## Verification

- **Explorer**: Check mint on [explorer.pyrachain.io](https://explorer.pyrachain.io)  
- **Balance**: Use `getBalance` from RPC guide  
- **Extensions**: Query mint account data for Token-2022 features

---

## Advanced Extensions

| Extension | Use Case | Import |
|-----------|----------|--------|
| **Transfer Fee** | Fee on transfers | `createInitializeTransferFeeConfigInstruction` |
| **Confidential Transfer** | Private balances | `createInitializeConfidentialTransferMintInstruction` |
| **Metadata Pointer** | On-chain name/symbol | `createInitializeMetadataPointerInstruction` |
| **Group Pointer** | Token groups | `createInitializeGroupPointerInstruction` |

See [SPL Token-2022 Docs](https://spl.solana.com/token-2022) for more.

---

## Tips & Security

- **Rent Exemption**: Always calculate to avoid account closure.  
- **Authority**: Revoke mint/freeze after initial minting for immutability.  
- **Errors**: Handle insufficient funds or invalid keys.  

---

**Mint. Extend. Innovate.**