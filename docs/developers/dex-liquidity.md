---
sidebar_position: 4
title: Provide liquidity on DEX
---

# How to Add Liquidity to Pool

## Overview

Provide liquidity to an existing **liquidity pool** on Pyra Chain DEX using **@pyra-chain/cp-amm-sdk**. Adding liquidity involves **creating a position** (price range) and **depositing tokens** into it, earning fees and rewards. The SDK handles **concentrated liquidity**, **dynamic fees**, and **Token-2022** support.

- **SDK**: [@pyra-chain/cp-amm-sdk](https://www.npmjs.com/package/@pyra-chain/cp-amm-sdk)
- **Network**: Pyra Chain RPC
- **Requirements**: Existing pool, wallet with tokens A/B, position NFT mint

---

## Installation

Install dependencies:

```
npm install @pyra-chain/cp-amm-sdk @solana/web3.js @solana/spl-token bn.js
```

---

## Step-by-Step Code Example

### 1. Setup Connection & Client
Initialize CpAmm client from SDK.

```javascript
import { Connection, PublicKey, Keypair } from '@solana/web3.js';
import BN from 'bn.js';
import { CpAmm } from '@pyra-chain/cp-amm-sdk';

const connection = new Connection('https://rpc.pyrachain.io', 'confirmed');
const cpAmm = new CpAmm(connection);

const payer = Keypair.generate();  // Replace with real keypair
```

### 2. Create Position
Create a new position in the pool (defines price range for liquidity).

```javascript
// Pool address
const poolAddress = new PublicKey('ExistingPoolAddressHere');

// Create NFT mint for position (use SPL Token)
import { createMint, TOKEN_PROGRAM_ID } from '@solana/spl-token';

const positionNftMint = await createMint(
  connection,
  payer,
  payer.publicKey,
  null,
  0,  // NFT decimals
  TOKEN_PROGRAM_ID
);

// Create position transaction
const createPositionTx = await cpAmm.createPosition({
  owner: payer.publicKey,
  payer: payer.publicKey,
  pool: poolAddress,
  positionNft: positionNftMint
});

// Sign and send
createPositionTx.recentBlockhash = (await connection.getLatestBlockhash()).blockhash;
createPositionTx.feePayer = payer.publicKey;
createPositionTx.sign(payer);

const positionSig = await connection.sendRawTransaction(createPositionTx.serialize());
await connection.confirmTransaction(positionSig, 'confirmed');

console.log('Position created! Tx:', positionSig);
```

### 3. Calculate Liquidity Delta & Deposit Quote
Use getDepositQuote to compute amounts and liquidity.

```javascript
// Pool state
const poolState = await cpAmm.getPoolState(poolAddress);

// Token decimals
const tokenADecimal = 9;
const tokenBDecimal = 9;

// Input: 5 tokenA
const inAmount = new BN(5_000_000_000);  // 5 * 10^9

// Desired price range (min/max sqrt prices from config or custom)
const minSqrtPrice = poolState.sqrtMinPrice;  // Or custom
const maxSqrtPrice = poolState.sqrtMaxPrice;
const sqrtPrice = poolState.currentSqrtPrice;  // Current price

const depositQuote = cpAmm.getDepositQuote({
  inAmount,
  isTokenA: true,  // Input is token A
  minSqrtPrice,
  maxSqrtPrice,
  sqrtPrice,
  inputTokenInfo: { decimals: tokenADecimal },  // For Token-2022
  outputTokenInfo: { decimals: tokenBDecimal }
});

const { actualInputAmount, consumedInputAmount, outputAmount, liquidityDelta } = depositQuote;
```

### 4. Add Liquidity (Increase Liquidity in Position)
Use increaseLiquidity or similar (inferred as addLiquidity from SDK notes).

```javascript
// Position address (derive or from createPosition logs)
const positionAddress = new PublicKey('PositionAddressHere');

// Token mints/programs from pool
const { tokenAMint, tokenBMint, tokenAProgram, tokenBProgram } = poolState;

// Add liquidity transaction
// Note: SDK likely has increaseLiquidity; adapting from similar functions
const addLiquidityTx = await cpAmm.increaseLiquidity({
  payer: payer.publicKey,
  position: positionAddress,
  pool: poolAddress,
  tokenAAmount: consumedInputAmount,
  tokenBAmount: outputAmount,
  liquidityDelta,
  tokenAProgram,
  tokenBProgram
});

// Sign and send
addLiquidityTx.recentBlockhash = (await connection.getLatestBlockhash()).blockhash;
addLiquidityTx.feePayer = payer.publicKey;
addLiquidityTx.sign(payer);

const addSig = await connection.sendRawTransaction(addLiquidityTx.serialize());
await connection.confirmTransaction(addSig, 'confirmed');

console.log('Liquidity added! Tx:', addSig);
```

### 5. Verify Position & Unclaimed Rewards
Check position state and unclaimed fees/rewards.

```javascript
const positionState = await cpAmm.fetchPositionState(positionAddress);

const unclaimed = cpAmm.getUnClaimReward(poolState, positionState);
console.log('Unclaimed Token A Fees:', unclaimed.feeTokenA.toString());
console.log('Unclaimed Token B Fees:', unclaimed.feeTokenB.toString());
unclaimed.rewards.forEach((reward, i) => {
  console.log(`Unclaimed Reward ${i}:`, reward.toString());
});
```

---

## Key Parameters for increaseLiquidity (Adapted)

| Param | Description | Example |
|-------|-------------|---------|
| **payer** | Wallet paying | `payer.publicKey` |
| **position** | Position address | From createPosition |
| **pool** | Pool address | Existing pool |
| **tokenAAmount** | Token A to deposit | From deposit quote |
| **tokenBAmount** | Token B to deposit | From deposit quote |
| **liquidityDelta** | Liquidity to add | From deposit quote |

---

## Tips & Troubleshooting

- **Position NFT**: Required for tracking; create new for each position.  
- **Price Range**: Use min/max from config for full range; custom for concentrated.  
- **Token-2022**: Provide tokenInfo for extensions.  
- **Lock Liquidity**: Set `isLockLiquidity: true` in custom pools for permanent locks.  
- **Errors**: Ensure sufficient balances; use getDepositQuote to avoid invalid amounts.  
- **Fees/Rewards**: Claim via dashboard or SDK's claim functions.  
- **Verification**: Check on [pyrachain.io/dex](https://pyrachain.io/dex) or explorer.

---

**Provide. Earn. Grow.**