---
slug: /burn-mechanism
sidebar_position: 3
title: Burn Mechanism
---

# Burn Mechanism

## Overview

The **$PYRA burn mechanism** is a core deflationary feature of the Pyra Chain ecosystem, designed to **permanently reduce circulating supply** across **both Solana and Pyra Chain networks simultaneously**. Every burn event is **synchronized in real time**, ensuring the 1:1 peg between $PYRA on Solana and $PYRA on Pyra Chain remains intact.

> **Goal**: Decrease total $PYRA in circulation → **increase scarcity** → **boost value per token**
> **Key Principle**: *The higher the network activity - the greater the burn - the stronger the token value.*

---

## Burn Sources

A portion of ecosystem fees contributes directly to burning. These fees are sent to a null address and removed from circulation **on both chains**.

| Source                  | Burn Rate               | Total Fee              |
|-------------------------|-------------------------|------------------------|
| **Launchpad Swaps**     | `1%` of each swap       | 2% (1% burn + 0.5% creator + 0.5% validators) |
| **DEX Swaps**           | `0.8%` of each swap     | 2% (0.8% burn + 0.5% creator + 0.5% validators + 0.2% protocol) |
| **Migration Fees**      | `5%` of bonding curve   | Split between burn and validators |
| **Token Creation Fees** | `100%` of creation fee  | 1,000 $PYRA (fully burned) |

---

## Synchronous Burning: How It Works

When a fee is generated on **either network**:

1. The fee is collected in $PYRA.
2. The **exact amount is burned on Pyra Chain**.
3. The **same amount is burned on Solana** via bridge protocol.
4. Supply decreases **identically** on both chains.
5. 1:1 peg preserved. No arbitrage. Full transparency.

This process is **on-chain** and **verifiable** in real time via the [Pyra Chain Explorer](https://explorer.pyrachain.io).

Real-time burn statistics are available at [pyrachain.io/vaults](https://pyrachain.io/vaults).