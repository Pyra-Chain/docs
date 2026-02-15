---
sidebar_position: 3
title: Create DEX pool
---

# How to Create DEX Pool

## Overview

Create a **liquidity pool** on the Pyra Chain DEX using the **@pyra-chain/cp-amm-sdk** (DAMM SDK). This SDK simplifies pool creation with **concentrated liquidity AMM** (CLMM) mechanics, supporting **Token-2022** and native SOL wrapping. Pools enable trading pairs with initial liquidity, dynamic fees, and position NFTs.

- **SDK**: [@pyra-chain/cp-amm-sdk](https://www.npmjs.com/package/@pyra-chain/cp-amm-sdk) 
- **Network**: Pyra Chain RPC
- **Requirements**: Wallet with $PYRA, token mints for pair (A/B)

---

## Installation

Install the DAMM SDK and dependencies:

```
npm install @pyra-chain/cp-amm-sdk @solana/web3.js @solana/spl-token bn.js
```

---

## Step-by-Step Code Example

### 1. Setup Connection & Import SDK
Connect to Pyra Chain and initialize the CpAmm client (from SDK).

```javascript
import { Connection, PublicKey, Keypair } from '@solana/web3.js';
import BN from 'bn.js';
import { CpAmm } from '@pyra-chain/cp-amm-sdk';  // Core AMM client
import { getSqrtPriceFromPrice } from '@pyra-chain/cp-amm-sdk';  // Price utils

// Connect to Pyra Chain RPC
const connection = new Connection('https://rpc.pyrachain.io', 'confirmed');

// Initialize CpAmm client
const cpAmm = new CpAmm(connection);

// Wallet payer (replace with your keypair)
const payer = Keypair.generate();  // Load real keypair in production
```

### 2. Prepare Pool Parameters
Fetch config, calculate initial liquidity/price, and get deposit quote.

```javascript
const configAddress = new PublicKey('CyBMMto3jsMhMGUjNPe61qgFK6orhmwjRQD8vbJjSDY4'); 

// Token mints (A: base, B: quote, e.g., $PYRA and another token)
const tokenAMint = new PublicKey('TokenAMintAddress');  // e.g., $PYRA mint
const tokenBMint = new PublicKey('TokenBMintAddress');
const tokenADecimal = 9;  // Decimals for token A
const tokenBDecimal = 9;  // Decimals for token B

// Token programs (Token-2022 or legacy)
const tokenAProgram = new PublicKey('TokenzQdBNbLqP5VEhdkAS6EPFLC1PHnBqCXEpPxuEb');  // Token-2022
const tokenBProgram = tokenAProgram;  // Same for both

// Initial price: 1 tokenA = 10 tokenB
const initPrice = 10;

// Get config state
const configState = await cpAmm.getConfigState(configAddress);

// Calculate sqrt price (Q64 format)
const sqrtPrice = getSqrtPriceFromPrice(initPrice, tokenADecimal, tokenBDecimal);

// Get deposit quote (5 tokenA input)
const { actualInputAmount, consumedInputAmount, outputAmount, liquidityDelta } = cpAmm.getDepositQuote({
  inAmount: new BN(5_000_000_000),  // 5 tokenA (9 decimals)
  isTokenA: true,
  minSqrtPrice: configState.sqrtMinPrice,
  maxSqrtPrice: configState.sqrtMaxPrice,  // Use max from config
  sqrtPrice: sqrtPrice,
  inputTokenInfo: { decimals: tokenADecimal },  // For Token-2022
  outputTokenInfo: { decimals: tokenBDecimal }
});
```

### 3. Create Position NFT Mint
Generate mint for the initial position NFT (tracks liquidity position).

```javascript
// Create mint for position NFT (use SPL Token functions)
import { createMint, TOKEN_PROGRAM_ID } from '@solana/spl-token';

// Assume you have a function to create mint (or use SDK if available)
const positionNftMint = await createMint(
  connection,
  payer,
  payer.publicKey,  // Mint authority
  null,  // Freeze authority
  0,  // Decimals for NFT
  TOKEN_PROGRAM_ID
);

console.log('Position NFT Mint:', positionNftMint.toBase58());
```

### 4. Create the Pool
Use `createPool` with prepared params. Locks initial liquidity if desired.

```javascript
// Activation point (null for immediate)
const activationPoint = null;

// Create pool transaction builder
const createPoolTx = await cpAmm.createPool({
  payer: payer.publicKey,
  creator: payer.publicKey,  // Can be different
  config: configAddress,
  positionNft: positionNftMint,
  tokenAMint,
  tokenBMint,
  activationPoint,
  tokenAAmount: consumedInputAmount,  // From quote
  tokenBAmount: outputAmount,
  initSqrtPrice: sqrtPrice,
  liquidityDelta,
  tokenAProgram,
  tokenBProgram,
  isLockLiquidity: false  // Set true to permanently lock position
});

// Build, sign, and send
const { blockhash } = await connection.getLatestBlockhash();
createPoolTx.recentBlockhash = blockhash;
createPoolTx.feePayer = payer.publicKey;
createPoolTx.sign(payer);  // Sign with payer (and others if needed)

const signature = await connection.sendRawTransaction(createPoolTx.serialize());
await connection.confirmTransaction(signature, 'confirmed');

console.log('Pool created! Tx:', signature);
```

### 5. Verify Pool Creation
Query the new pool state.

```javascript
// Get pool address (derive from params or from tx logs)
const poolAddress = new PublicKey('YourPoolAddressFromLogs');  // Extract from signature

const poolState = await cpAmm.getPoolState(poolAddress);
console.log('Pool Initialized:', poolState.initialized);
console.log('Pool Sqrt Price:', poolState.currentSqrtPrice.toString());
```
---

## Key Parameters Explained

| Param | Description | Example |
|-------|-------------|---------|
| **payer** | Wallet paying fees | `wallet.publicKey` |
| **creator** | Pool owner | `wallet.publicKey` |
| **config** | Pool config account | From ecosystem config |
| **positionNft** | Initial NFT mint | Newly created mint |
| **tokenAMint/tokenBMint** | Trading pair mints | $PYRA and custom token |
| **initSqrtPrice** | Starting price (Q64) | From `getSqrtPriceFromPrice` |
| **liquidityDelta** | Initial liquidity | From `getDepositQuote` |
| **isLockLiquidity** | Permanent lock? | `false` for editable |

---

## Tips & Troubleshooting

- **SOL Wrapping**: SDK auto-wraps native tokens.  
- **Token-2022**: Provide `inputTokenInfo`/`outputTokenInfo` for extensions.  
- **Errors**: Check for zero amounts or invalid prices. Use `getDepositQuote` to validate.  
- **Fees**: Dynamic (volatility-based) + base fee from config.  
- **Verification**: View pool on [pyrachain.io/dex](https://pyrachain.io/dex) or explorer.  
- **Docs**: Full SDK at [npmjs.com/@pyra-chain/cp-amm-sdk](https://www.npmjs.com/package/@pyra-chain/cp-amm-sdk).

---

**Pool. Trade. Earn.**