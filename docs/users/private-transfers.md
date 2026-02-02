---
sidebar_position: 6
title: Private Transfers
---

# Private Transfers

## Anonymous $PYRA Transactions

**Private Transfers** is an optional privacy feature in the Pyra Chain wallet that **breaks the link between sender and recipient**. By routing transfers through a mixing pool with fixed denominations, your transaction history remains confidential.

- **Access**: Toggle in wallet during transfer
- **Method**: Pool mixing with fixed 100K $PYRA denomination
- **Maximum**: 2.5M $PYRA per transfer
- **Result**: Sender and recipient addresses are unlinkable
- **Network**: Pyra Chain only

---

## How Private Transfers Work

### The Mixing Process

When you enable Private Transfers:

1. **Deposit**: Your $PYRA enters a shared mixing pool
2. **Fixed Denomination**: Amount is processed in 100K $PYRA units (max 2.5M)
3. **Pool Mixing**: Your tokens mix with others in the pool
4. **Withdrawal**: Recipient receives fresh $PYRA from the pool
5. **Broken Link**: No on-chain connection between sender and recipient

```
Sender → [Mixing Pool] → Recipient
         (fixed denominations)
```

---

## How to Send a Private Transfer

### Step 1: Open Wallet & Start Transfer
Access your wallet and initiate a transfer.

**Action**:
1. Visit [pyrachain.io/wallet](https://pyrachain.io/wallet)
2. Connect/unlock your wallet
3. Click **"Send"** or **"Transfer"**

---

### Step 2: Enable Private Transfer
Toggle the privacy option before entering amount.

**Action**:
1. Find **"Private Transfer"** toggle/checkbox
2. Enable it (switch to ON)
3. Notice: Available denominations may be shown

---

### Step 3: Enter Transfer Details
Specify recipient and amount.

**Action**:
1. Enter **Recipient Address**
2. Enter **Amount** (will be rounded to fixed denominations)
3. Review available denominations
4. System shows actual amount after denomination matching

---

### Step 4: Confirm & Send
Complete the private transfer.

**Action**:
1. Review transfer details:
   - Recipient address
   - Amount (in fixed denominations)
   - Privacy fee (if applicable)
2. Click **"Send Private Transfer"**
3. Approve transaction in your Solana wallet
4. Wait for confirmation

---

## Fixed Denomination & Limits

Private transfers use a single standardized denomination to ensure maximum anonymity:

| Parameter | Value |
|-----------|-------|
| **Fixed Denomination** | **100,000 $PYRA** |
| **Maximum Transfer** | **2,500,000 $PYRA** (25 units) |

> **Note**: Your transfer amount must be in multiples of 100K $PYRA. For example, 500,000 $PYRA = 5 units of 100K.

---

## Privacy Guarantees

| Feature | Benefit |
|---------|---------|
| **No Direct Link** | Sender and recipient addresses cannot be connected on-chain |
| **Fixed Amounts** | Standard denominations prevent amount-based tracking |
| **Pool Mixing** | Multiple users' funds mix together |
| **Optional** | Use only when privacy is needed |

---

## When to Use Private Transfers

- **Personal Privacy**: Keep your financial activity confidential
- **Business Transactions**: Protect sensitive payment information
- **Security**: Prevent tracking of your wallet holdings
- **Donations**: Anonymous contributions to causes

---

## Tips & Best Practices

- **Use Standard Amounts**: Transfers matching exact denominations process faster
- **Check Pool Liquidity**: Larger denominations may have longer wait times
- **Verify Recipient**: Double-check address - private transfers are irreversible
- **Regular Transfers Available**: Use standard transfers when privacy isn't needed
- **No Extra KYC**: Privacy is a native feature, not a separate service

---

## Limitations

- **Pyra Chain Only**: Private transfers work within Pyra Chain network
- **Fixed Denomination**: Only 100K $PYRA units supported
- **Maximum**: 2.5M $PYRA per transfer (25 units)
- **Pool Dependent**: Requires liquidity in mixing pool

---

**Transfer. Mix. Stay Private.**
