---
sidebar_position: 6
title: NPM packages
---

# Our NPM Packages

## Build on Pyra Chain with Official SDKs

The Pyra Chain ecosystem provides **open-source TypeScript SDKs** for seamless integration with our DEX, bonding curves, and liquidity protocols. These packages are **fully audited**, **TypeScript-native**, and optimized for **41K TPS** performance. Install via npm and start building dApps, pools, and tokens today.

All packages are **public** (v1.0+), with comprehensive docs and examples.

---

## Available Packages

| Package | Version | Description | Install |
|---------|---------|-------------|---------|
| **@pyrachain/damm-sdk** | [1.0.0](https://www.npmjs.com/package/@pyrachain/damm-sdk) | **DAMM SDK** for creating and managing **concentrated liquidity pools** on Pyra Chain DEX. Supports **Token-2022**, dynamic fees (volatility-based), base fee schedulers (linear/exponential), and position NFTs. Key functions: `createPool`, `getDepositQuote`, `getUnClaimReward`. Ideal for DeFi dApps and LP providers. | `npm install @pyrachain/damm-sdk` |
| **@pyrachain/dynamic-bonding-curve-sdk** | [1.0.3](https://www.npmjs.com/package/@pyrachain/dynamic-bonding-curve-sdk) | **Dynamic Bonding Curve SDK** for launching tokens with customizable curves, vesting, and migration options. Features **config creation** (`createConfig`), fee params (`getDynamicFeeParams`), locked vesting (`getLockedVestingParams`), and quote reserves. Perfect for fair token distributions. | `npm install @pyrachain/dynamic-bonding-curve-sdk` |

---

## Quick Start

### DAMM SDK Example (Create Pool)
```javascript
import { CpAmm } from '@pyrachain/damm-sdk';

const cpAmm = new CpAmm(connection);  // Pyra Chain RPC
const createPoolTx = await cpAmm.createPool({
  payer: wallet.publicKey,
  tokenAMint: new PublicKey('...'),
  tokenBMint: new PublicKey('...'),
  // ... other params
});
```

### Bonding Curve SDK Example (Create Config)
```javascript
import { createConfig } from '@pyrachain/dynamic-bonding-curve-sdk';

const tx = await createConfig({
  payer: wallet.publicKey,
  config: new PublicKey('...'),
  poolFees: { /* fee config */ },
  // ... other params
});
```

---

## Resources

- **Docs**: Full API in package READMEs (linked above)
- **RPC**: Use [rpc.pyrachain.io](/developers/rpc) for all integrations
- **Support**: [X @pyrachain](https://x.com/pyrachain)

**Build on Pyra Chain today.**