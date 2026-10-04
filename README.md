# bWalletX Server

An open-source BSV indexer for **1Sat Ordinals** and **BSV-21 tokens** that doesn't need a full node.

It talks directly to the BSV network, keeps only block headers, reads each block once for ordinal and token outputs, and then discards the block. All it stores is the index, so it runs on an ordinary server instead of beside a 7 TB+ node.

It is being built as an independent backend for [bWalletX](https://bwalletx.com), alongside the existing 1Sat and WhatsOnChain services, and is free for anyone to run.

**Status:** planning. See [PLAN.md](PLAN.md).

## Built on

- [bitcoinX](https://github.com/kyuupichan/bitcoinX) by Neil Booth: networking, headers, transactions and script.
- The design of [ElectrumX](https://github.com/kyuupichan/electrumx), also by Neil Booth: an async server that indexes the chain for light wallets.

## Licence

MIT, see [LICENSE](LICENSE). bitcoinX is a dependency under its own Open BSV licence.
