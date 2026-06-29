---
title: "FAQ"
weight: 7
---

# FAQ

## What is Ethwei?

Ethwei is a Cosmos SDK proof-of-stake blockchain. It is intentionally simple: fixed emissions,
no community tax, no smart contracts (yet), no complex governance.

## Is this mainnet?

No. Ethwei is currently testnet (`ethwei-testnet-1`). Testnet tokens have no monetary value.

## When is mainnet?

No date is set. Mainnet will launch once the testnet is sufficiently stable and the validator
set is mature enough.

## What wallet should I use?

[Keplr](https://www.keplr.app/) is the recommended wallet. See the
[Wallet guide]({{< relref "wallet.md" >}}) to add Ethwei Testnet in one click.

## What is WEI?

`WEI` is the minimal denomination of ETE. `1 ETE = 1,000,000 WEI`. Transactions on-chain use
`WEI`; wallets and explorers display `ETE`.

## Why coinType 118?

Ethwei uses Cosmos's standard coin type (ATOM's coin type). This means standard Cosmos HD
derivation paths work — your Keplr/Ledger seed generates Ethwei addresses the same way it
generates Cosmos Hub addresses.

## What is the community tax?

0%. All block rewards go to validators and delegators.

## Will the emission schedule change?

No. The 100M ETE/year for 30 years is fixed in genesis. There is no governance mechanism to
change it. This is a feature, not a bug.

## Where is the source code?

Check [github.com/ethwei](https://github.com/ethwei) — repository links will be published here
as they go public.

## I found a bug / I want to validate — how do I reach you?

Community channels will be listed here once the testnet is open. For now, check the main site
at [ethwei.com](https://ethwei.com).
