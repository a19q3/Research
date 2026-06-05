# Lerna Mapping From Morph Channel

Status: adaptation notes  
Date: 2026-06-05

## Purpose

This document maps Morph Channel concepts to a Cardano/Hydra-native Lerna design. The mapping is intentionally not mechanical. When CKB has a primitive with no clean Cardano equivalent, the mismatch is stated directly.

## Core Mapping Table

| Morph / CKB concept | Lerna / Cardano-Hydra equivalent | Notes and mismatch |
| --- | --- | --- |
| Stable funding bundle / vault Cells | Cardano UTxOs deposited into a Hydra Head, or a compact factory UTxO inside the Head | In V0 the Head is the stable outer value container. Lerna should not imitate CKB vault Cells on L1. A compact factory UTxO inside the Head is the closest functional object. |
| Moving State Cell | Lerna `FactoryState` object inside the Head, plus signed `LatestStateHeader` evidence carried in Head-local transactions | CKB uses a live State Cell pointer. Hydra uses snapshots over a Head UTxO set. Lerna's state pointer is Head-local, not a global L1 Cell. |
| `PUBLISH` | Submit or checkpoint a Lerna state into the open Head, then have it included in a Hydra snapshot | V0 publication is ordinary Hydra Head progress. There is no separate L1 publication transaction for Lerna. |
| `SUPERSEDE` | A newer valid Lerna state supersedes an older Lerna state at the factory layer; at the Head layer, a newer Hydra snapshot contests an older close | Supersession must respect Hydra's close/contest mechanism. Lerna cannot bypass Hydra contestation in V0. |
| `FINALIZE` | Materialise compact Lerna rights into ordinary Head UTxOs, then allow Hydra fanout after the contestation deadline | Finality is split: Lerna materialises inside the Head; Hydra finalises the Head state on L1. |
| CKB `since` challenge window | Hydra contestation period and contestation deadline, plus a Lerna watcher response profile | CKB relative `since` is not available as a direct Lerna primitive. In V0, stale Lerna evidence is defeated by contesting the Head with a newer snapshot. |
| Sponsor partition | Operator fee budget, Head-level transaction fee budget, and optional Lerna materialisation budget | Cardano fees and min-ADA are different from CKB capacity fees. The safety idea is to keep fee budgets separate from virtual-channel balances. |
| xUDT conservation | ADA-only V0 conservation first; future native-asset profile by policy ID and asset name | Cardano native assets are multi-asset values in UTxOs, but Hydra native-token fanout hazards make them an extension track, not a first V0 claim. |
| Factory proof mode | Access manifest, touched leaves, roots, rights-dependency graph, and `NonInterferenceProof` inside `FactoryState` transitions | Merkle membership is not sufficient. The proof must cover indirect rights through reserve, membership, and exit paths. |
| Settlement descriptor | Cardano output materialisation descriptor for local exit, rebalance, splice, fanout preparation, and optional V1 claim UTxO | The descriptor must derive valid Cardano outputs with address, value, datum, reference script expectations, and min-ADA. |
| Stable channel identity | `(network_id, hydra_head_id, factory_id, factory_epoch)` | `factory_epoch` changes after re-anchoring or major reserve restructure. |
| Funding epoch / vault set | Factory epoch plus materialised reserve set inside the Head | Cardano has no CKB-style vault set. The closest equivalent is the set of Head UTxOs or compact factory reserve commitments. |
| Payload commitment | Factory roots and virtual-channel state hashes | Lerna separates balances, subchannels, membership, reserve, and release receipts into distinct roots. |
| Plaintext bilateral witness mode | Simple virtual-channel state for V0 payment channels | A bilateral ADA-only prototype should stay plaintext for auditability. Factory mode can be commitment-first. |
| Envelope-first factory authorisation | `AccessManifest.transition_family` and `body_commitment` before proof interpretation | This maps cleanly. It is one of Morph's strongest ideas and should be preserved. |
| Touched leaves | Leaves read, written, inserted, removed, or proven absent in Lerna roots | The touched set must include reserve and membership dependencies, not only balances. |
| Non-interference | Proof that untouched participants' rights are unchanged under the declared transition family | This becomes Lerna's central safety condition for reduced signing sets. |
| Partition conservation | Per-lane conservation for lovelace, min-ADA reserve, native assets, factory reserve, and operator budget | Cardano's multi-asset model makes this natural but unforgiving. Policy ID and asset name are the identity, not ticker metadata. |
| Sponsored publication package | Watcher/operator package for Head-local materialisation and Hydra contest response | In V0 this is operational rather than a new script-level sponsor object. |
| Exact materialisation | Descriptor-derived Head UTxOs must exactly match value, address, datum, and reference-script expectations | The Cardano analogue is stricter because output size and min-ADA can make a theoretically valid settlement unspendable or unfannable. |

## Design Modes

### Mode V0: Conservative Hydra-native Mode

V0 keeps Lerna entirely inside an open Hydra Head. The first credible version is ADA-only.

Properties:

- Lerna state is represented by a compact Head-local factory object.
- Virtual-channel updates may be off-chain, but settlement and materialisation occur inside the Head.
- A Head close is handled only through Hydra close, contest, and fanout.
- Every unresolved virtual right must be materialised, released, or fully authorised before fanout.
- No new L1 dispute protocol is introduced.
- Non-Head virtual participants do not gain an independent L1 claim path.
- Reduced-signature non-interference falls back to full factory authorisation unless the graph predicate is implemented and benchmarked.

Why this is the first POC path:

- It uses Hydra's existing contestation model.
- It avoids untested L1 validators for factory adjudication.
- It lets developers test compact factory accounting and materialisation under real Head-local eUTxO constraints.
- It aligns with Cardano community expectations that Hydra applications should first respect the Head lifecycle.

Main V0 risk:

- If participants wait too long before materialisation, the final Head snapshot may contain too many outputs or too-large datums to handle safely.
- If authorised Head-side actors censor or fail, a non-Head virtual participant may be unable to force the latest Lerna state into a contesting snapshot.

### Mode V1: Robust Fanout-aware Mode

V1 allows unresolved Lerna factory state to fan out as a script-controlled L1 claim UTxO.

Properties:

- The final Hydra UTxO set may contain a Lerna claim UTxO.
- Aiken/Plutus validators enforce latest-state settlement, local exits, non-interference proofs, and asset conservation.
- Virtual-channel participants can claim after Head fanout without needing all virtual channels pre-materialised.

Why this is future work:

- The validator must verify signatures, state numbers, roots, non-interference, release receipts, and Cardano output construction.
- Proof sizes may exceed practical Plutus budgets.
- min-ADA and native-asset bundle constraints are easy to under-specify.
- L1 claim UTxOs create a new dispute surface that must not conflict with Hydra's own safety guarantees.

## Morph Concepts That Should Not Be Ported Directly

### CKB State Cell As L1 Pointer

Cardano does not have CKB Cells with type-script identity and `since`-controlled moving state. A Lerna V0 state pointer should live inside the Head. If it is later moved to L1 in V1, it must be a Cardano script-controlled UTxO with its own datum and redeemer semantics, not a simulated CKB Cell.

### CKB `since`

Hydra already has a contestation period and deadline. Lerna V0 should use watcher response budgets relative to Hydra's deadline. Recreating CKB-style `since` inside the Head would confuse the design.

### Sponsor Capacity

Cardano transaction fees and min-ADA do not behave like CKB capacity. The useful abstraction is a separated operator or close budget, not a literal sponsor Cell partition.

### xUDT Typing

Cardano native assets are identified by policy ID and asset name. Do not map xUDT type scripts to token names or metadata. The policy ID is part of the asset identity.

## Lerna-specific Additions Beyond Morph

### Compact Factory UTxO

Lerna's compact factory UTxO is a Hydra-native object. It reduces Head-internal UTxO explosion during normal operation and delays expansion until local exit or shutdown preparation.

### Materialisation Discipline

Lerna treats materialisation as a protocol phase, not as an implementation detail. This is the key difference from a generic off-chain ledger inside Hydra.

### Release Receipts

Release receipts record that a compact right has been replaced by a materialised output or closed with zero value. They prevent the same virtual right from surviving in both compact and materialised form.

### Snapshot-aware Watcher Profile

Lerna watchers track two sequences: virtual-channel latest states and Hydra snapshots. A newer virtual state is not enough in V0 unless it can be reflected in a newer Hydra snapshot before the contestation deadline.

### Rights-dependency Graph

Non-interference is defined over rights, not only leaves. A balance leaf may depend on shared reserve, membership, exit eligibility, and fee-budget claims. The graph makes those indirect dependencies explicit.

## Minimal POC Mapping

For the first implementation, use:

- ADA-only balances;
- two or three Head participants;
- one compact factory UTxO inside the Head;
- one `balances_root`;
- one `release_receipt_root`;
- plaintext touched leaves;
- no V1 claim UTxO;
- local materialisation before close;
- deterministic prefix materialisation;
- tests for stale exit, newer-state materialisation, receipt replay, same-number equivocation, and Head-side censorship.

Then extend to:

- native assets by policy ID and asset name;
- min-ADA reserve accounting;
- subchannel roots;
- non-interference proofs;
- rebalances;
- partial materialisation;
- Hydrozoa-style flexible custody experiments.
