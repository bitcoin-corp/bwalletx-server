# bWalletX Server: plan

## Goal

Index 1Sat Ordinals and BSV-21 tokens on BSV **without running a full node**, and answer the questions a wallet asks:

- What ordinals and tokens does this address hold?
- What is this token (ticker, supply, icon, issuer)?
- What is listed for sale, and at what price?
- Where is this inscription now, and what is its content?

Success means bWalletX can switch its token and ordinal lookups to this server and get the same answers it gets from 1Sat today.

## Why no full node

The BSV chain is several terabytes and growing. A full node costs more than this project can carry. The data a wallet needs (ordinal and token outputs and their spends) is a tiny fraction of it.

## How it works

```
BSV peers ──P2P──▶ headers (tens of MB, kept)
                 └▶ blocks, one at a time ──▶ parse ──▶ keep ordinal/token outputs + spends ──▶ index (LevelDB/SQLite)
                                                    └─▶ block discarded
index ──▶ HTTP API (1Sat-compatible shapes) ──▶ wallets
```

1. **Headers.** bitcoinX syncs and validates the header chain from peers and handles reorgs.
2. **Blocks.** Fetch each block from the start height, the first 1Sat inscription in 2023, to the tip, then follow new blocks and the mempool.
3. **Filter fast.** Most transactions hold no ordinals. Do a cheap byte scan for the inscription envelope and for spends of outputs we already track, and fully parse only the hits.
4. **Index.** Store inscriptions, BSV-21 deploys/mints/transfers and OrdLock listings, keyed by outpoint, address and token id.
5. **Serve.** A small async HTTP API whose responses use the same shape as the 1Sat endpoints bWalletX uses, so switching is a configuration change.

## Milestones

| # | Milestone | Done when | Estimate |
|---|---|---|---|
| 0 | **Feasibility test** | Fetched a few hundred recent blocks via bitcoinX; measured MB/block, blocks/min and parse speed; extracted inscriptions from them | 1 day |
| 1 | **Block feed** | Headers synced; blocks streamed in order from the start height; resumes after restart; survives a reorg | 3–5 days |
| 2 | **Ordinals** | Inscriptions recorded with content type and origin; ownership followed through sends and sales; OrdLock listings and cancels tracked | 3–5 days |
| 3 | **BSV-21 tokens** | Deploy+mint, transfer and burn rules applied exactly as 1Sat does; balances per address per token | 1–2 weeks |
| 4 | **API** | Address holdings, token details, inscription lookup and content, market listings, in 1Sat-compatible shapes | 3–5 days |
| 5 | **Parity check** | Run beside 1Sat; compare holdings for a large sample of addresses until they agree; publish the comparison | 1–2 weeks (overlaps sync) |
| 6 | **bWalletX switch-over** | bWalletX can use this server as primary or fallback, chosen in settings | 2–3 days |

**Total build:** about 3–6 weeks of development, plus the first catch-up sync (weeks, unattended). Milestone 0 turns these estimates into measured numbers.

## Running costs

- **Server:** one Hetzner machine with a few hundred GB of disk, roughly £40–60 per month.
- **Bandwidth:** the first catch-up downloads every block since 2023 once, possibly several TB. Hetzner bandwidth is unmetered. After that it's just the chain's daily growth.
- **No full node.**

## Risks and open questions

- **Peers serving old blocks.** Ordinary BSV nodes may serve historical blocks slowly or not at all. Fallback: fetch raw blocks from a public block source such as JungleBus or WhatsOnChain.
- **Parse speed in Python.** Large blocks may be slow to read. Mitigations: the byte-scan filter, parallel workers, or a small native helper for the scan.
- **BSV-21 rule parity.** Token validity rules must match 1Sat's exactly or balances will differ. Milestone 5 exists to catch this.
- **Chain growth since 2023.** Not yet measured; milestone 0 measures it.

## Principles

- **Open source** (MIT) from the first commit.
- **Wallet-first:** index only what wallets need.
- **Replaceable:** compatible API shapes, so any wallet can point at any instance.
- **No keys, no custody.** The server only reads the public chain.

## Credits

Built on Neil Booth's bitcoinX, and following the design of his ElectrumX.
