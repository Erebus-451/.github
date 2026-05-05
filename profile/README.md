---

<div align="center">

# 𝐁𝐮𝐢𝐥𝐝𝐢𝐧𝐠 𝐄𝐑𝐂-𝟒𝟓𝟏 — 𝐭𝐡𝐞 𝐬𝐭𝐚𝐧𝐝𝐚𝐫𝐝 𝐭𝐡𝐞𝐲 𝐝𝐨𝐧'𝐭 𝐰𝐚𝐧𝐭 𝐲𝐨𝐮 𝐭𝐨 𝐡𝐚𝐯𝐞

[erebus.build](https://erebus.build) · [Twitter](https://x.com/Erebus_build) · [GitHub](https://github.com/Erebus-451)

</div>

---

## What is ERC-451?

ERC-404 was experimental. Promising. Broken. Six critical bugs — gas explosions, a live reentrancy vector, broken marketplace discovery, a supply counter that never decreased. No fixes. It was buried.

**We dug it up.**

ERC-451 is a rebuilt semi-fungible token standard. Every whole ERC-20 unit corresponds to one ERC-721 NFT. Buying and selling fractions automatically mints and burns NFTs. Named after HTTP 451: *Unavailable For Legal Reasons.*

---

## What we fixed

| Bug | Fix | Gas saving |
|-----|-----|-----------|
| CRIT-01: O(N) transfer loop | `_batchTransferFromOwned` — single pass | 55–65% |
| CRIT-02: double SSTORE per ownership update | `_setOwnerAndIndex` — one packed write | 25–35% |
| CRIT-03: no reentrancy guard on safeTransferFrom | `nonReentrant` covers full callback | — |
| CRIT-04: supply counter never decreases | `erebusCirculatingSupply()` = minted − banked | — |
| CRIT-05: supportsInterface returns false for IERC721 | Reports IERC721 + IERC721Metadata | — |
| CRIT-06: exemption toggle hits block gas limit | Paginated with `batchSize` parameter | >98% |

Additional: `uint32[]` owned arrays (8× denser), dedicated mint path, EIP-2612 permit, EIP-4906 MetadataUpdate, parameterised custom errors.

---

## $EREBUS

1,000 pixelated caskets on Ethereum mainnet. 1 $EREBUS = 1 NFT — native liquidity, native fractionalization. No wrappers. No middlemen.

```
Mainnet:  0x9fE4Ad1BA6993012Cd457e2C39f867F45dBFd8fc
```

---

## ErebusGamble

Provably-fair 1v1 NFT dueling powered by Chainlink VRF v2.5.

- Both players stake their NFT into escrow
- Chainlink assigns each a verifiable random roll (1–100)
- Higher roll wins both NFTs — 0% house edge, fully on-chain

---

## Repos

| Repo | Description |
|------|-------------|
| [Erebus-451/erc451](https://github.com/Erebus-451/erc451) | Open source ERC-451 standard |
| [Erebus-451/erebus-labs](https://github.com/Erebus-451/erebus-labs) | Full monorepo — contracts, web, API |

---

## Quick start

```bash
git clone https://github.com/Erebus-451/erebus-labs.git
cd erebus-labs
npm install
npm run dev
```

Copy `.env.example` files in `packages/contracts/`, `apps/web/`, and `apps/api/` before running.

---

## Structure

```
erebus-labs/
├── packages/contracts/     # ERC451.sol · Erebus.sol · ErebusGamble.sol
├── apps/web/               # React + Vite + wagmi
└── apps/api/               # Express + Prisma + duel indexer
```

---

<div align="center">

*451: Unavailable For Legal Reasons.*

</div>
