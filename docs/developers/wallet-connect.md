---
sidebar_position: 5
title: Wallet Connect
---

# Wallet Connect Integration

## Connect Solana Wallets to Your dApp

Integrate **Wallet Connect** to allow users to connect their Solana wallets (Phantom, Solflare, Backpack, etc.) to your dApp on Pyra Chain. Users sign transactions through their existing wallet - no custom key management required.

- **Method**: Solana Wallet Adapter
- **Wallets**: Phantom, Solflare, Backpack, and more
- **Signing**: All transactions signed via connected wallet
- **Network**: Works on Pyra Chain (Solana-compatible)

---

## Installation

Install the Solana Wallet Adapter packages:

```bash
npm install @solana/wallet-adapter-base @solana/wallet-adapter-react @solana/wallet-adapter-react-ui @solana/wallet-adapter-wallets @solana/web3.js
```

---

## Basic Setup (React)

### 1. Configure Wallet Providers

Set up the wallet context with supported wallets.

```tsx
import { FC, ReactNode, useMemo } from 'react';
import { ConnectionProvider, WalletProvider } from '@solana/wallet-adapter-react';
import { WalletModalProvider } from '@solana/wallet-adapter-react-ui';
import {
  PhantomWalletAdapter,
  SolflareWalletAdapter,
  BackpackWalletAdapter,
} from '@solana/wallet-adapter-wallets';

// Pyra Chain RPC endpoint
const PYRA_RPC = 'https://rpc.pyrachain.io';

export const WalletContextProvider: FC<{ children: ReactNode }> = ({ children }) => {
  const wallets = useMemo(
    () => [
      new PhantomWalletAdapter(),
      new SolflareWalletAdapter(),
      new BackpackWalletAdapter(),
    ],
    []
  );

  return (
    <ConnectionProvider endpoint={PYRA_RPC}>
      <WalletProvider wallets={wallets} autoConnect>
        <WalletModalProvider>
          {children}
        </WalletModalProvider>
      </WalletProvider>
    </ConnectionProvider>
  );
};
```

### 2. Add Connect Button

Use the built-in wallet button or create a custom one.

```tsx
import { WalletMultiButton } from '@solana/wallet-adapter-react-ui';

// Import default styles
import '@solana/wallet-adapter-react-ui/styles.css';

export const Navbar: FC = () => {
  return (
    <nav>
      <WalletMultiButton />
    </nav>
  );
};
```

### 3. Access Wallet in Components

Use hooks to access the connected wallet.

```tsx
import { useWallet, useConnection } from '@solana/wallet-adapter-react';

export const WalletInfo: FC = () => {
  const { publicKey, connected, disconnect } = useWallet();
  const { connection } = useConnection();

  if (!connected) {
    return <p>Please connect your wallet</p>;
  }

  return (
    <div>
      <p>Connected: {publicKey?.toBase58()}</p>
      <button onClick={disconnect}>Disconnect</button>
    </div>
  );
};
```

---

## Signing Transactions

### Send Transaction

```tsx
import { useWallet, useConnection } from '@solana/wallet-adapter-react';
import { Transaction, SystemProgram, PublicKey, LAMPORTS_PER_SOL } from '@solana/web3.js';

export const SendTransaction: FC = () => {
  const { publicKey, sendTransaction } = useWallet();
  const { connection } = useConnection();

  const handleSend = async (recipient: string, amount: number) => {
    if (!publicKey) return;

    const transaction = new Transaction().add(
      SystemProgram.transfer({
        fromPubkey: publicKey,
        toPubkey: new PublicKey(recipient),
        lamports: amount * LAMPORTS_PER_SOL,
      })
    );

    const { blockhash } = await connection.getLatestBlockhash();
    transaction.recentBlockhash = blockhash;
    transaction.feePayer = publicKey;

    // Wallet prompts user to sign
    const signature = await sendTransaction(transaction, connection);

    // Wait for confirmation
    await connection.confirmTransaction(signature, 'confirmed');

    console.log('Transaction confirmed:', signature);
  };

  return (
    <button onClick={() => handleSend('RecipientAddressHere', 0.1)}>
      Send 0.1 $PYRA
    </button>
  );
};
```

### Sign Message

```tsx
import { useWallet } from '@solana/wallet-adapter-react';

export const SignMessage: FC = () => {
  const { publicKey, signMessage } = useWallet();

  const handleSign = async () => {
    if (!publicKey || !signMessage) return;

    const message = new TextEncoder().encode('Sign this message for Pyra Chain');
    const signature = await signMessage(message);

    console.log('Signature:', Buffer.from(signature).toString('base64'));
  };

  return <button onClick={handleSign}>Sign Message</button>;
};
```

---

## Svelte Integration

For SvelteKit projects, use the wallet adapter stores.

```bash
npm install @solana/wallet-adapter-base @solana/web3.js
```

```svelte
<script lang="ts">
  import { onMount } from 'svelte';
  import { Connection, PublicKey } from '@solana/web3.js';

  let wallet: any = null;
  let publicKey: string | null = null;

  const PYRA_RPC = 'https://rpc.pyrachain.io';
  const connection = new Connection(PYRA_RPC, 'confirmed');

  const connectWallet = async () => {
    // Check for Phantom
    if ('phantom' in window) {
      const provider = (window as any).phantom?.solana;
      if (provider?.isPhantom) {
        const response = await provider.connect();
        wallet = provider;
        publicKey = response.publicKey.toString();
      }
    }
  };

  const disconnectWallet = async () => {
    if (wallet) {
      await wallet.disconnect();
      wallet = null;
      publicKey = null;
    }
  };
</script>

{#if publicKey}
  <p>Connected: {publicKey}</p>
  <button on:click={disconnectWallet}>Disconnect</button>
{:else}
  <button on:click={connectWallet}>Connect Wallet</button>
{/if}
```

---

## Check Connection Status

```ts
import { useWallet } from '@solana/wallet-adapter-react';

const { connected, publicKey, wallet } = useWallet();

// Check if connected
const isConnected = connected && publicKey;

// Get wallet name
const walletName = wallet?.adapter.name; // "Phantom", "Solflare", etc.

// Get public key as string
const address = publicKey?.toBase58();
```

---

## Supported Wallets

| Wallet | Adapter | Install |
|--------|---------|---------|
| **Phantom** | `PhantomWalletAdapter` | `@solana/wallet-adapter-wallets` |
| **Solflare** | `SolflareWalletAdapter` | `@solana/wallet-adapter-wallets` |
| **Backpack** | `BackpackWalletAdapter` | `@solana/wallet-adapter-wallets` |
| **Ledger** | `LedgerWalletAdapter` | `@solana/wallet-adapter-wallets` |

---

## Best Practices

| Rule | Why |
|------|-----|
| **Use autoConnect** | Reconnect returning users automatically |
| **Handle errors** | Catch wallet rejection and network errors |
| **Show wallet state** | Display connected address in UI |
| **Confirm transactions** | Wait for on-chain confirmation |
| **Use Pyra RPC** | Connect to `rpc.pyrachain.io` for Pyra Chain |

---

## Resources

- **Pyra RPC**: [rpc.pyrachain.io](https://rpc.pyrachain.io)
- **Wallet Adapter Docs**: [solana-labs/wallet-adapter](https://github.com/solana-labs/wallet-adapter)
- **Support**: [X @pyrachain](https://x.com/pyrachain)

---

**Connect. Sign. Build.**
