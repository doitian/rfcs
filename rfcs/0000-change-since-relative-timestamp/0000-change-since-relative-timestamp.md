---
Number: "0000"
Category: Consensus (Hard Fork)
Status: Draft
Author: Ian Yang
Organization: Nervos Foundation
Created: 2021-02-03
---

# Use input cell committing block timestamp as the start time for the relative timestamp in `since`

## Abstract

This document proposes a transaction verification consensus change. When the `since` field of the transaction input uses a relative timestamp, the referenced cell committing block timestamp is used instead of the median timestamp.

This is a modification to the RFC17 [Transaction valid since](../0017-tx-valid-since/0017-tx-valid-since.md).

## Motivation

The current consensus rule uses the median of the timestamp fields of the 37 blocks preceding the referenced cell committing block. It is resource consuming to get the median timestamp because it requires either getting 37 block headers or caching the median timestamp for each block. The intention to use the median time is to avoid miners profitable by manipulating block timestamp to include more transactions, but it is safe to use the committing block timestamp as the start time because of two reasons:

1. The timestamp in the block header has already been verified by the network that it must be larger than the median of the previous 37 blocks and less than or equal to the current time plus 15 seconds. TODO: reference to block data structure RFC
2. The transaction consuming a cell with the `since` requirement is committed later than the transaction creating the cell. When miners see the consuming transaction, the start timestamp has already been determined. 

## Specification

When an input `since` field is present, and

* The `metric_flag` is block timestamp (10).
* The `relative_flag` is relative (1).

The transaction is mature when

```
StartTime + SinceValue ≥ MedianTimestamp
```

where

* `StartTime` is the block timestamp that committed the cell consumed by the input.
* `SinceValue` is the `value` part of the `since` field.
* `MedianTimestamp` is the median timestamp of the previous 37 blocks preceding the block if the transaction is in the block, or the latest 37 blocks if the transaction is in the pool.

The transaction verification fails if the transaction is immature.

The only change is `StartTime`, which is the median of the previous 37 blocks preceding the one that has committed the consumed cell. Because block timestamp must be larger than the median of its previous 37 blocks, the new consensus rule is looser than the old rule. The transaction that is mature using the old rule must be mature using the new rule, but the transaction that is mature using the new rule may be immature using the old rule.

## Test Vectors

Following is an example that a transaction is mature using the new rule but is immature using the old rule.

Assuming that:

* A transaction consumes a cell in block S and is about to be committed into block T with a since requirement that:
	* The `metric_flag` is block timestamp (10).
	* The `relative_flag` is relative (1).
	* The `value` is 600,000 (10 minutes).
* The median of the previous 37 blocks preceding block S is 10,000.
* The timestamp of block S is 20,000.
* The median of the previous 37 blocks preceding block T is 615,000

In the old consensus, StartTime + SinceValue = 10,000 + 600,000 = 610,000, which is less than the MedianTimestamp 615,000.

But in the new rule, StartTime + SinceValue = 20,000 + 600,000 = 620,000 ≥ 615,000.

## Deployment

The deployment can be performed in two stages.

The first stage will activate the new consensus rule starting from a specific epoch. The mainnet and testnet will use different starting epochs and all other chains will use the new rule from epoch 0.

After the fork is activated, the old rule will be replaced by the new rule starting from the genesis block in all chains.

## Backward compatibility

Because the new consensus rule is looser than the old one, this proposal must be deployed via a hard fork.
