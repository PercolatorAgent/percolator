# Position NFT Wrapper

Transferable Token-2022 NFTs representing open Percolator perpetual futures positions.

## Overview

[`dcccrypto/percolator-nft`](https://github.com/dcccrypto/percolator-nft) is an external wrapper program that mints NFTs backed by open positions in Percolator slabs. It reads position state directly from slab account data (no CPI into core) and uses SPL Token-2022 with `decimals=0, supply=1` per position.

## Architecture

```
percolator-nft (wrapper program)
  ├── Reads position state from Percolator slab accounts (CPI-free, direct data read)
  ├── SPL Token-2022 (mint/burn position NFTs)
  ├── PositionNft PDA (links NFT mint → slab + user_idx)
  └── SettleFunding (permissionless crank to sync funding index before transfer)
```

### Why a wrapper?

- **Core stays lean** — no Token-2022 dependency in the Percolator BPF binary
- **Independent upgradability** — iterate on NFT logic without touching the engine
- **Security isolation** — NFT bugs cannot affect core funds or margin calculations
- **Same pattern** as the staking wrapper (`percolator-stake`)

## Instructions

| Tag | Name | Description |
|-----|------|-------------|
| 0 | `MintPositionNft` | Mint an NFT for an open position (caller must own the position) |
| 1 | `BurnPositionNft` | Burn the NFT, release position back to direct ownership |
| 2 | `SettleFunding` | Permissionless crank — update funding index before transfer |

## PDA Seeds

- **PositionNft**: `["position_nft", slab_pubkey, user_idx_le_bytes]`
- **MintAuthority**: `["mint_authority"]` (program-wide, signs all mint operations)

## Transfer Hook (Future)

A Token-2022 `TransferHook` extension will enforce that funding is settled before any NFT transfer, preventing stale-funding exploits when positions change hands. The hook invokes `SettleFunding` automatically during `transfer_checked`.

## Security

- `forbid(unsafe_code)` enforced
- Slab owner verified against known Percolator program IDs
- Position ownership verified before minting
- NFT burn closes PDA and returns rent to holder

## Repository

**Source:** [github.com/dcccrypto/percolator-nft](https://github.com/dcccrypto/percolator-nft)
