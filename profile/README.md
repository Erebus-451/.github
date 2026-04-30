<div align="center">

# EREBUS LABS

**Building ERC-451 — the standard they don't want you to have**

[erebus.build](https://erebus.build) · [Twitter](https://x.com/Erebus_build)

</div>

---

## The problem

ERC-404 was experimental. Promising. Broken.

Gas costs were brutal — a 10-token transfer could cost 10× what it should. A reentrancy vulnerability sat open in `safeTransferFrom`. Marketplaces couldn't recognise the NFTs because `supportsInterface` returned the wrong values. The circulating supply counter never decreased on burn. The exemption toggle would hit the block gas limit on any LP pool with a meaningful balance.

Six critical bugs. No fixes. The standard was quietly buried.

**We dug it up.**

---

## What we built

### ERC-451

A leaner, cheaper, safer semi-fungible token standard. Every ERC-20 whole unit corresponds to one ERC-721 NFT — buying and selling fractions automatically mints and burns NFTs. The design is identical to ERC-404 at the surface. The implementation is rebuilt from scratch.

Named after HTTP 451: *Unavailable For Legal Reasons.*

| Bug | Fix |
|---|---|
| CRIT-01: per-token loop on transfer | Single-pass `_batchTransferFromOwned` — 92–95% cheaper on multi-token transfers |
| CRIT-02: two SSTOREs per ownership update | `_setOwnerAndIndex` packs owner + array index into one `uint256`, single SSTORE |
| CRIT-03: no reentrancy guard on `safeTransferFrom` | `nonReentrant` lock covers the full call including `onERC721Received` callback |
| CRIT-04: `erc721TotalSupply()` never decreases | `erc721CirculatingSupply()` returns `minted - bankLength` — true live count |
| CRIT-05: `supportsInterface` returns false for IERC721 | Reports IERC721 + IERC721Metadata — fully visible to OpenSea, Blur, etc. |
| CRIT-06: exemption toggle hits block gas limit | Paginated `_setERC721TransferExempt(address, bool, batchSize)` |

Additional improvements: `uint32[]` owned arrays (8× denser than ERC-404's `uint256[]`), dedicated `_mintERC20` path skipping the cold `address(0)` SLOAD, arithmetic post-transfer balance derivation (no re-reads), EIP-2612 `permit()`, EIP-4906 `MetadataUpdate`, parameterised custom errors throughout.

### $EREBUS

5,000 pixelated caskets on Ethereum mainnet. One whole `$EREBUS` = one NFT — native liquidity, native fractionalization. No wrappers. No bridges. No middlemen.

Built on ERC-451, the token includes a launch safety trading gate: `tradingEnabled` defaults to `false`, allowing the deployer to seed liquidity before opening trading to the public. Once enabled, it cannot be reversed.

### ErebusGamble

A provably-fair 1v1 NFT dueling platform powered by **Chainlink VRF v2.5**.

- Two players stake their NFTs into escrow
- Each player funds an independent VRF request (paid in native ETH — no LINK required)
- Chainlink assigns each player a verifiable random roll (1–100)
- Higher roll wins. Challenger wins on tie. Winner claims both NFTs.
- 0% house edge. No admin influence on outcomes. Fully on-chain.

---

## Repositories

| Repo | Description |
|---|---|
| [Erebus-451/erc451](https://github.com/Erebus-451/erc451) | Open source ERC-451 standard — the base contract |
| [Erebus-451/erebus-gamble](https://github.com/Erebus-451/erebus-gamble) | ErebusGamble — Chainlink VRF v2.5 dueling contract |
| [ClaimyToken/erebus-labs](https://github.com/ClaimyToken/erebus-labs) | Full project monorepo (contracts, web app, API) |

---

## Monorepo structure

```
erebus-labs/
├── packages/
│   └── contracts/          # Solidity contracts + Hardhat
│       ├── contracts/
│       │   ├── ERC451.sol       # The standard
│       │   ├── Erebus.sol       # Token contract (inherits ERC451)
│       │   └── ErebusGamble.sol # VRF dueling contract
│       ├── scripts/
│       │   ├── deployErebus.ts  # Deploy token only
│       │   ├── deployGamble.ts  # Deploy dueling contract
│       │   └── launch.ts        # setupLiquidityPair + enableTrading
│       └── test/
│           └── Erebus.test.ts   # 20 tests
└── apps/
    ├── web/                # React + Vite + wagmi frontend
    └── api/                # Express API + Prisma + duel indexer
```

---

## Running locally

### Prerequisites

- Node.js 18+
- npm or pnpm

### 1. Clone and install

```bash
git clone https://github.com/ClaimyToken/erebus-labs.git
cd erebus-labs
npm install
```

### 2. Environment variables

**Contracts** — copy `packages/contracts/.env.example` to `packages/contracts/.env`:

```env
SEPOLIA_RPC_URL=https://...
MAINNET_RPC_URL=https://...
DEPLOYER_PRIVATE_KEY=...
ETHERSCAN_API_KEY=...
```

**Web** — copy `apps/web/.env.example` to `apps/web/.env`:

```env
VITE_COMING_SOON_PASSWORD=...   # Beta gate password (optional — bypassed if unset)
```

**API** — copy `apps/api/.env.example` to `apps/api/.env`:

```env
DATABASE_URL=...
SEPOLIA_RPC_URL=...
GAMBLE_DEPLOY_BLOCK=...
```

### 3. Compile contracts

```bash
cd packages/contracts
npx hardhat compile
```

### 4. Run tests

```bash
npx hardhat test
```

### 5. Start the web app

```bash
cd apps/web
npm run dev
```

### 6. Start the API

```bash
cd apps/api
npm run dev
```

---

## Deployment

### Token (Erebus)

```bash
cd packages/contracts

# Sepolia testnet
npx hardhat run scripts/deployErebus.ts --network sepolia

# Mainnet
npx hardhat run scripts/deployErebus.ts --network mainnet
```

### Launch (after adding liquidity on Uniswap V2)

```bash
EREBUS_ADDRESS=0x... PAIR_ADDRESS=0x... npx hardhat run scripts/launch.ts --network mainnet
```

This calls `setupLiquidityPair`, then `enableTrading` — fully opens the token to public trading.

### Dueling contract (ErebusGamble)

```bash
npx hardhat run scripts/deployGamble.ts --network sepolia
```

Post-deploy: add the contract as a VRF consumer at [vrf.chain.link](https://vrf.chain.link), then call `enableStake(true)`.

---

## Technical notes

**Why `address(0)` is always exempt**

The trading gate checks `erc721TransferExempt(from) || erc721TransferExempt(to)`. Since `address(0)` is always exempt, mints (`from = address(0)`) are never blocked regardless of `tradingEnabled`. This allows initial supply distribution before trading opens.

**Why ErebusGamble must NOT be set ERC-721-transfer-exempt**

The exemption flag blocks inbound NFT transfers. ErebusGamble takes physical custody of NFTs via `transferFrom` — if exempt, all deposits would revert with `RecipientIsERC721TransferExempt`.

**VRF race condition**

Chainlink may fulfil the challenger's VRF request before an opponent accepts. ErebusGamble handles this: the roll is stored but resolution is gated on `state == Active`. When the opponent's VRF fulfils later, both conditions are met and the duel resolves in that callback.

**`uint32[]` owned arrays**

Solidity packs `uint32` values 8-per-slot. ERC-404 used `uint256[]` — 1-per-slot. For a 500-NFT holder, reading the full owned array costs ~62 SLOADs in ERC-451 vs ~500 in ERC-404.

---

<div align="center">

[erebus.build](https://erebus.build) · [Twitter](https://x.com/Erebus_build)

*451: Unavailable For Legal Reasons.*

</div>
