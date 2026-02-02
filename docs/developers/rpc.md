---
sidebar_position: 1
title: RPC
---

# How to Connect to RPC

## Overview

The **Pyra Chain RPC** provides a **high-throughput endpoint** for developers to interact with the Pyra Chain network, compatible with **Solana's JSON-RPC API**. Use it for querying balances, sending transactions, and more - all at **41K TPS** with low latency.

- **HTTP Endpoint**: [rpc.pyrachain.io](https://rpc.pyrachain.io)
- **WebSocket Endpoint**: [ws.pyrachain.io](wss://ws.pyrachain.io) (for real-time subscriptions)
- **Compatibility**: Full Solana RPC methods (e.g., `getBalance`, `sendTransaction`).

---

## Installation

First, install the Solana Web3.js library in your project:

```bash
npm install @solana/web3.js
```

---

## Basic Connection Example

Connect to the HTTP RPC for simple queries. Use the `Connection` class from `@solana/web3.js`.

```javascript
import { Connection, PublicKey } from '@solana/web3.js';

// Initialize connection to Pyra Chain RPC
const connection = new Connection(
  'https://rpc.pyrachain.io',  // HTTP endpoint
  'confirmed'  // Commitment level: 'confirmed' for speed
);

// Example: Get balance of a public key
async function getBalance(pubkeyString) {
  try {
    const publicKey = new PublicKey(pubkeyString);
    const balance = await connection.getBalance(publicKey);
    console.log(`Balance: ${balance} lamports`);
    return balance;
  } catch (error) {
    console.error('Error fetching balance:', error);
  }
}

// Usage
getBalance('YourWalletAddressHere');  // Replace with actual address
```

**Output Example**:  
```
Balance: 5000000000 lamports  // ~5 $PYRA
```

---

## WebSocket Connection for Subscriptions

For real-time updates (e.g., account changes, new blocks), use the WebSocket endpoint.

```javascript
import { Connection, PublicKey } from '@solana/web3.js';

// Initialize with WebSocket
const connection = new Connection(
  'https://rpc.pyrachain.io',  // Primary HTTP
  {
    commitment: 'confirmed',
    wsEndpoint: 'wss://ws.pyrachain.io'  // WebSocket for subscriptions
  }
);

// Subscribe to account changes
async function subscribeToAccountChanges(pubkeyString) {
  const publicKey = new PublicKey(pubkeyString);
  
  connection.onAccountChange(
    publicKey,
    (accountInfo) => {
      console.log('Account updated:', accountInfo);
    },
    'confirmed'  // Commitment level
  );
  
  console.log('Subscribed to account changes');
}

// Usage
subscribeToAccountChanges('YourWalletAddressHere');
```

**Notes**:  
- WebSocket auto-reconnects on failure.  
- Use for logs, new signatures, or slot updates.  

---

## Advanced Usage

### Send a Transaction
```javascript
import { Connection, Transaction, SystemProgram, LAMPORTS_PER_SOL } from '@solana/web3.js';
import { Keypair } from '@solana/web3.js';  // For signing

const connection = new Connection('https://rpc.pyrachain.io');

// Example: Transfer 0.1 $PYRA
async function sendTransaction(fromKeypair, toPubkeyString) {
  const toPublicKey = new PublicKey(toPubkeyString);
  
  const transaction = new Transaction().add(
    SystemProgram.transfer({
      fromPubkey: fromKeypair.publicKey,
      toPubkey: toPublicKey,
      lamports: 0.1 * LAMPORTS_PER_SOL,
    })
  );
  
  const signature = await connection.sendTransaction(transaction, [fromKeypair]);
  console.log('Transaction signature:', signature);
  
  // Confirm
  await connection.confirmTransaction(signature);
  console.log('Transaction confirmed!');
}

// Usage (use test keys)
const fromKeypair = Keypair.generate();  // Replace with real keypair
sendTransaction(fromKeypair, 'RecipientAddressHere');
```

### Query Recent Blockhash
```javascript
async function getRecentBlockhash() {
  const { blockhash } = await connection.getLatestBlockhash();
  console.log('Recent Blockhash:', blockhash);
  return blockhash;
}
```

---

## Error Handling & Best Practices

- **Commitment Levels**: `'processed'` (fastest), `'confirmed'`, `'finalized'` (safest).  
- **Security**: Never expose private keys in client-side code. Use server-side for signing.  
- **Testing**: Use [explorer.pyrachain.io](https://explorer.pyrachain.io) to verify transactions.  

---

## Previous & Next

**Connect. Build. Deploy.** 

Join [Telegram](https://t.me/PyraChannel) for dev support.