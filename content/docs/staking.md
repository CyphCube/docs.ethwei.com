---
title: "Staking"
weight: 3
---

# Staking

Staking ETE secures the network and earns you a share of block rewards. There is no community
tax — 100% of emissions go to validators and delegators.

## Prerequisites

- Keplr installed and Ethwei Testnet added ([Wallet guide]({{< relref "wallet.md" >}}))
- Some testnet ETE in your wallet

---

## Delegate via Keplr

1. Open Keplr and switch to **Ethwei Testnet**.
2. Click **Stake** in the Keplr dashboard.
3. Select a validator from the list.
4. Enter the amount of ETE to delegate and confirm.
5. Approve the transaction in Keplr.

Rewards begin accruing immediately at each block.

---

## Delegate via Explorer

1. Go to [explorer.ethwei.com/testnet](https://explorer.ethwei.com/testnet).
2. Navigate to **Staking → Validators**.
3. Click a validator, then **Delegate**.
4. Connect Keplr when prompted, enter your amount, and submit.

---

## Withdraw rewards

**Via Keplr:** Open Keplr → Stake → **Claim Rewards**.

**Via Explorer:**
1. Go to your account page on the explorer.
2. Click **Claim Rewards** and approve in Keplr.

Rewards are paid in ETE and land in your wallet immediately.

---

## Unbond (undelegate)

Unbonding returns your staked ETE to your wallet after the **unbonding period**.

> [!WARNING]
> **Unbonding period:** Once you start unbonding, your ETE is locked for the unbonding period
> and earns no rewards during that time. Check the chain parameters on the explorer for the
> current unbonding duration.

**Via Explorer:**
1. Go to **Staking → Validators** and select your validator.
2. Click **Undelegate**, enter the amount, and confirm.

**Via Keplr:**
1. Open Keplr → Stake → your validator → **Undelegate**.

---

## Redelegate

To move stake from one validator to another without waiting for unbonding:

1. Explorer → your validator → **Redelegate**.
2. Choose the destination validator, enter the amount, confirm.

Redelegation takes effect immediately. Note: redelegated stake cannot be redelegated again for
the unbonding period (the "redelegation stacking" rule).
