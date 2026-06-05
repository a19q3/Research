# Lerna Open Questions

Status: research backlog  
Date: 2026-06-05

## P0: Questions Blocking A Credible V0 Prototype

### Who Can Force Inclusion In V0?

V0 must name the actor that can move newer Lerna evidence into a Hydra snapshot:

- Head participant acting for itself;
- operator who is also a Head participant;
- delegated watcher with a live submission path to a Head participant.

Blocking issue:

- A non-Head virtual participant has no independent L1 claim in V0. If no authorised actor can checkpoint or materialise on their behalf, the security claim must be downgraded to assisted/cooperative.

### What Is The Minimal Factory State Encoding?

Need to choose the smallest useful V0 state:

- ADA-only balances;
- factory state number;
- participant registry;
- virtual-channel map;
- release receipt set;
- settlement descriptor hash.

Open issue:

- Whether to put full data in an inline datum for easy testing or store roots with off-chain proof material from the beginning.

### How Does Lerna Check State Monotonicity Inside A Head?

V0 needs a Head-local rule that rejects lower or equal factory state numbers.

Options:

- script-controlled compact factory UTxO with datum transition checks;
- all-participant signatures over Head-local transactions;
- application-level Hydra node plugin logic before a validator exists.

Preferred first step:

- an Aiken or Plutus validator for the compact factory UTxO, with a deliberately small datum and ADA-only accounting.

### What Is The Exact Materialisation Rule?

Need a deterministic function:

```text
FactoryState + SettlementDescriptor + TouchedLeaves -> Set<CardanoOutput>
```

The function must define:

- address;
- lovelace;
- min-ADA;
- datum;
- reference script expectations;
- release receipt creation;
- ordering of outputs for test determinism.

V0 is ADA-only. Native assets move to the extension profile after the ADA path has measured output count, transaction size, datum size, min-ADA footprint, and time-to-materialise.

Required batch semantics:

- commit a conservative `BudgetProfile`;
- compute deterministic `Fits_P(k)`;
- materialise exactly the maximum fitting prefix;
- persist the suffix in `pending_materialisation_root`;
- define what happens if the Head closes between batches.

### How Are Release Receipts Stored And Consumed?

Open issue:

- Whether release receipts are leaves in the factory state, standalone Head UTxOs, or signatures carried in materialisation witnesses.

Safety requirement:

- A compact right must not remain claimable after its replacement output has been materialised.
- `receipt_id` must bind network id, Head id, factory id, factory epoch, right id, state number, materialisation plan hash, batch index, output index, and replacement output hash.
- The receipt must be absent from the old receipt root, present in the new receipt root, and tied to removal of the active compact right.

### What Is The Watcher Obligation In V0?

Need a precise liveness profile:

- observe latest virtual-channel signatures;
- observe Head snapshots;
- detect stale materialisation or stale Head close;
- prepare Head-local materialisation transaction;
- contest stale Head snapshot before the Hydra contestation deadline.

Open issue:

- Whether virtual participants who are not Head participants can rely on delegated watchers without weakening safety claims.

### What Are The Required Failure Scenarios?

The V0 prototype should explicitly test or simulate:

- operator refuses checkpoint;
- Head participant censors materialisation;
- watcher is late;
- proof material is withheld after a high-number header is signed;
- same-number equivocation;
- materialisation output count exceeds the budget profile;
- receipt replay;
- native-asset bundle is rejected under `ledger_profile = V0-ADA`.

## P1: Questions For Factory Proof Mode

### What Commitment Scheme Should Be Used?

Candidates:

- sparse Merkle trees;
- Merkle Patricia-like maps;
- KZG commitments;
- Hydra accumulator-inspired approaches;
- simple sorted-map hashes for V0 tests.

Evaluation criteria:

- proof size;
- Aiken/Plutus verification cost;
- ease of off-chain proof construction;
- ability to prove absence;
- compatibility with partial materialisation.

### What Is The Rights-dependency Graph?

Need a formal schema for:

- balance rights;
- reserve claims;
- membership rights;
- local exit rights;
- fee-budget claims;
- release receipts;
- virtual-channel app state.

The graph must answer:

- which untouched rights might be affected by a touched leaf;
- when reduced signing sets are safe;
- when full factory authorisation is required.

### Can Non-interference Be Modular?

Open issue:

- Whether each transition family can define its own local non-interference predicate, or whether the factory must use one global proof system.

Risk:

- Per-transition predicates may become inconsistent and allow authority confusion.

Conservative rule:

- the access manifest must declare the transition family and body commitment before proof interpretation.

## P1: Hydra Interaction Questions

### When Must Materialisation Begin Before Head Close?

Need a deployment parameter based on:

- expected number of virtual channels;
- expected number of materialised outputs;
- transaction size;
- native-asset bundle size;
- Head participant responsiveness;
- Hydra contestation period.

The protocol should define a warning threshold and a hard shutdown-preparation threshold.

### Can Lerna Use Partial Fanout Safely?

Hydra supports partial fanout, but Lerna should not assume this solves all cases.

Need tests for:

- many small ADA-only outputs;
- outputs with inline datums;
- native-token outputs;
- script-controlled outputs;
- failed final partial fanout.

### How Does Lerna Interact With Deposit And Decommit?

Need to distinguish:

- Head-level deposit/decommit;
- Lerna factory reserve increase/decrease;
- virtual-channel rebalance;
- virtual-channel local exit.

Risk:

- A user may think a Head decommit is a virtual-channel withdrawal. The API must make the boundary explicit.

### Can A Lerna Factory Survive Head Membership Changes?

Hydra Head membership is currently a fixed practical assumption. Hydrozoa explores dynamic membership.

Open questions:

- Does `factory_epoch` advance on membership change?
- Are virtual channels with departing Head participants force-materialised?
- Can a virtual participant remain while their Head representative changes?

## P2: V1 Claim-UTxO Research

### What Must The L1 Claim Validator Enforce?

Minimum checks:

- domain-separated signatures;
- factory identity and epoch;
- state monotonicity;
- membership proofs;
- touched leaves;
- non-interference;
- asset conservation;
- min-ADA-safe outputs;
- release receipts;
- bounded transition family.

Open issue:

- Whether this can fit within Plutus execution and transaction-size limits.

### What Is The L1 Claim UTxO Datum?

Possible contents:

- full factory roots;
- asset registry hash;
- reserve hash;
- descriptor hash;
- highest observed state number;
- timeout profile;
- unresolved channel root.

Risk:

- Too much datum makes the output expensive or unfannable; too little datum shifts too much work to redeemers.

### How Are Competing Claims Ordered?

If two participants submit different latest states, the validator needs a clear rule:

- higher state number wins;
- same state number requires identical state hash or is treated as equivocation;
- lower state number is rejected;
- invalid transition family is rejected before proof body interpretation.

Open issue:

- How to handle withholding of proof material for a higher signed state.

### Does V1 Need Its Own Contestation Period?

If unresolved factory state reaches L1, Hydra's contestation period has already ended. V1 may need an inner claim deadline or response window.

Risk:

- This recreates a second dispute protocol and must be justified carefully.

## Evaluation Backlog

### Test Matrix

Build tests for:

- cooperative update;
- stale virtual-channel state;
- stale Head close contested by newer snapshot;
- local exit with full authorisation;
- local exit with non-interference proof;
- failed non-interference proof;
- release receipt double-use;
- ADA-only conservation;
- native-asset conservation as extension-only;
- min-ADA underfunding;
- large datum materialisation;
- too-many-output materialisation.

### Metrics

Measure:

- Head-local transaction size;
- validator execution units;
- proof byte size;
- number of outputs after materialisation;
- time to materialise before close;
- watcher response time;
- partial fanout success rate;
- storage overhead for virtual-channel histories.

### Developer Ergonomics

Prototype API concepts:

- `createFactory`;
- `openVirtualChannel`;
- `submitVirtualUpdate`;
- `checkpointFactory`;
- `rebalance`;
- `localExit`;
- `materialise`;
- `prepareForFanout`;
- `closeFactory`.

Need to decide:

- whether this belongs as a library around Hydra node, a Hydra Pay-like service, or a standalone Lerna node.

## Non-goals For The First Paper

Do not solve:

- global routing;
- liquidity discovery;
- privacy networks;
- watchtower markets;
- cross-Head atomic routing;
- production validator optimisation;
- governance token mechanics;
- replacement of Hydra close/contest/fanout.

## Recommended Next Milestones

1. Write a minimal formal V0 state transition specification.
2. Build an ADA-only Aiken validator sketch for compact factory state.
3. Run a local Hydra demo with two virtual channels and materialisation before close.
4. Add release receipt tests.
5. Add native-asset and min-ADA conservation tests.
6. Only then revisit V1 claim-UTxO design.
