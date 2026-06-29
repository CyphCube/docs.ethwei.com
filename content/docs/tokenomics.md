---
title: "Tokenomics"
weight: 4
---

# Tokenomics

Ethwei's token model is designed to be predictable and boring. No complex vesting cliffs,
no governance-controlled inflation, no surprises.

## Token: ETE

| Property | Value |
|---|---|
| Ticker | **ETE** |
| Minimal denom | `WEI` |
| Decimals | 6 (1 ETE = 1,000,000 WEI) |
| Genesis supply | 7,000,000,000 ETE (7B) |
| Maximum supply | 10,000,000,000 ETE (10B) |
| Annual emission | 100,000,000 ETE / year (100M) |
| Emission period | 30 years |
| Block emission | ~15.84 ETE / block |
| Community tax | **0%** |

## Supply schedule

The chain mints a fixed **100,000,000 ETE per year** for 30 years, starting at genesis.
This is not inflation-rate based — it is a fixed absolute amount, independent of total supply.

```
Year 0    7,000,000,000 ETE (genesis)
Year 1    7,100,000,000 ETE
Year 5    7,500,000,000 ETE
Year 10   8,000,000,000 ETE
Year 20   9,000,000,000 ETE
Year 30  10,000,000,000 ETE (maximum supply reached)
```

After 30 years, emissions stop. The chain continues with transaction fees only.

## Block rewards

At ~6.3-second block times (CometBFT default):

```
Blocks per year ≈ 5,000,000
Emission per block ≈ 100,000,000 / 5,000,000 = 20 ETE
```

> [!NOTE]
> The spec says ~15.84 ETE/block, which corresponds to ~6,313,131 blocks/year
> (closer to ~5-second average block times). Your actual block reward depends on
> observed block time on the live chain.

## Distribution

All block rewards go to **validators and their delegators** proportional to stake.
There is no community pool tax, foundation cut, or protocol fee.

```
Block reward
  └─ 100% to validator set
       ├─ Validator commission (set individually per validator)
       └─ Delegator rewards (proportional to stake)
```

## Denomination note

On-chain, all amounts are in `WEI` (the minimal denom). When you see a transaction for
`1000000 WEI`, that is `1 ETE`. Wallets and explorers display the `ETE` denom for readability.
