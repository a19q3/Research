# Lerna Protocol: Hydra-native Virtual Channel Factories for Cardano eUTxO

Status: revised technical design note; ADA-only V0 scope  
Date: 2026-06-05  
Scope: research design, not a production specification

## Abstract

Lerna Protocol is a proposed design note and research agenda for a virtual-channel factory layer in Hydra-family Cardano eUTxO systems. It adapts ideas from Morph Channel, namely stable value authority, moving signed state evidence, sponsored publication boundaries, partitioned conservation, local factory proofs, touched leaves, and non-interference, without treating Cardano as if it were CKB. The result is not "eltoo for Cardano", not a replacement for Hydra, and not a Hydrozoa competitor. It is a compact discipline for running many virtual channels inside, or alongside, Hydra-family Heads.

The central thesis is that Hydra already supplies the hard outer machinery: fast isomorphic eUTxO execution, multi-party snapshots, close, contest, and fanout. Lerna supplies an inner factory semantics: a way to represent many virtual channels as a compact Head-local state object, to update them with latest-state signatures, to materialise ordinary Cardano outputs before Hydra fanout, and eventually to prove local exits without disturbing untouched participants. The concrete V0 target in this note is ADA-only and remains entirely inside an open Hydra Head. A research V1 mode allows unresolved factory state to fan out as a script-controlled L1 claim UTxO, where Aiken or Plutus validators could enforce latest-state settlement and non-interference proofs.

Lerna is deliberately cautious. It preserves Cardano-native terms such as Hydra Head, snapshot, contestation deadline, fanout, eUTxO, min-ADA, reference input, and script-controlled UTxO. It uses eltoo only as background for non-punitive latest-state settlement. It treats Hydrozoa's dynamic and rule-based directions as compatibility surfaces rather than rivals. The first claim is not a deployable protocol; it is an ADA-only V0 state machine, receipt anti-replay rule, deterministic materialisation discipline, and failure model suitable for a prototype.

## 1. Introduction And Motivation

Hydra Head is the first practical member of the Hydra family. It lets a fixed set of participants run Cardano-like transactions off-chain with low latency, confirm state in signed snapshots, and return the final UTxO set to L1 through close, contest, and fanout. Recent Hydra implementation work also emphasises a simpler lifecycle: Heads can open empty and receive funds through deposit/increment, avoiding an older initialisation phase with commit and abort complexity. This makes Hydra a natural substrate for application-specific L2 protocols, but it also exposes an economic and operational question: must every bilateral channel, balance, rebalance, and local exit become a Head-visible UTxO?

Lerna answers no. It proposes a Head-local factory state that can hold many virtual-channel rights compactly, while retaining deterministic paths back to ordinary Cardano outputs. The protocol is aimed at wallets, payment applications, app-specific liquidity groups, and dApps that want the speed of Hydra but do not want to represent every temporary virtual channel as a full Head UTxO during normal operation.

The source inspiration is Morph Channel, a CKB paper that separates stable funding, moving state evidence, sponsor-owned publication, partition conservation, and factory-local proofs. A direct port would be a mistake. CKB Cells, type scripts, lock scripts, `since`, and xUDT do not map one-to-one to Cardano. Cardano has eUTxO, native assets, Plutus/Aiken validators, inline datums, reference inputs, script-controlled UTxOs, min-ADA, transaction validity intervals, and Hydra's own Head state machine. Lerna therefore recasts Morph's abstractions as Hydra-native objects. Because Morph is a local research draft by the same author, it is treated here as design stimulus, not as an externally verifiable security foundation.

The practical problem is not only throughput. It is exit discipline. A virtual channel factory is useful only if the participants can tell, at every point, what would happen if the outer Head stopped making progress. Lerna therefore treats materialisation as a first-class protocol object. Before a Head fans out, Lerna V0 must expand every unresolved virtual right into ordinary Head UTxOs or agreed zero-value releases. Lerna V1 may instead fan out a compact claim object, but only if a real validator design can enforce latest-state settlement and non-interference on L1.

## 2. Why Not "Eltoo For Cardano"

The eltoo lineage is useful but narrow. It gives one important intuition: a newer mutually authorised state should supersede stale evidence without penalty. That intuition is not enough to define a Cardano protocol.

Lerna is not eltoo for three reasons.

First, Hydra already has a dispute surface. A Head closes with a snapshot; participants may contest with a newer snapshot; after the contestation deadline, fanout distributes the resolved UTxO set. Lerna must fit into this close/contest/fanout mechanism rather than invent a parallel L1 replacement game for V0.

Second, Cardano's accounting is not Bitcoin's output scripting model. Even the ADA-only V0 path must account for lovelace, min-ADA, script datums, reference inputs, and validator resource limits; a later multi-asset profile must also account for native assets by policy ID and asset name. A "newer state wins" slogan does not define how a compact factory state materialises into valid eUTxO outputs.

Third, the Cardano community already has Hydra-family research directions. Interhead Hydra studies composition across Heads. Hydrozoa explores a flexible multi-party state-channel design that decouples execution from custody and has a rule-based fallback regime. Lerna should be positioned as an inner virtual-channel factory layer that can live inside these systems, not as a replacement narrative.

The correct positioning is:

- Hydra provides the outer L2 state channel and snapshot/fanout machinery.
- Hydrozoa-like systems may provide more flexible custody, dynamic membership, and fallback regimes.
- Lerna provides factory-local semantics for virtual channels, latest-state headers, release receipts, non-interference proofs, and materialisation plans.

## 3. Related Work

### Hydra Head

Hydra Head is an isomorphic multi-party state channel for Cardano. Participants deposit UTxOs, run Cardano-like transactions off-chain, confirm state in snapshots, close with a snapshot if needed, contest stale snapshots, and fan out the resolved state back to L1. The implementation and documentation are directly relevant for Lerna because they define the outer lifecycle and the practical limits that Lerna must respect: participant limits, transaction size limits, native-token fanout hazards, deposit/decommit mechanics, snapshot numbering, contestation deadlines, and partial fanout.

The most important Lerna consequence is that V0 should not add a new L1 dispute protocol. It should behave as a Head-local application whose compact state is unfolded before fanout.

### Interhead Hydra

Interhead Hydra studies communication and value transfer across Heads. Lerna is compatible with this direction because a Lerna virtual channel need not always be a bilateral relation inside one static Head. It can be an inner relation anchored in one Head, or eventually a virtual relation whose endpoints are Head-local representatives. That said, this paper does not solve cross-Head routing or liquidity discovery.

### Hydrozoa

Hydrozoa is a Cardano multi-party state-channel project that explicitly separates execution from custody and discusses dynamic deposits, withdrawals, and rule-based fallback. Its current public materials mark the implementation as pre-alpha and the old specification as superseded by a new whitepaper in progress. Lerna should therefore use Hydrozoa cautiously: as evidence that the Cardano community is exploring flexible Head-family custody and fallback regimes, not as a fixed target to depend upon.

The compatibility point is strong. Lerna's factory state can be interpreted as an L2 ledger object inside a Hydrozoa-style head. Hydrozoa's rule-based regime also resembles Lerna V1's future claim-UTxO direction, although Lerna does not claim that this validator design is complete.

### eltoo

eltoo is relevant as an intellectual ancestor for non-punitive latest-state settlement. Lerna adopts the monotonic-state-number idea and rejects penalty-based settlement as the default. It does not adopt Bitcoin-specific sighash assumptions, transaction replacement mechanisms, or output templates.

### Channel Factories And Perun

Channel factories show why shared funding can support many off-chain relationships. Perun and related virtual-channel work show the broader adjudication pattern: off-chain state becomes enforceable through a dispute or adjudication mechanism. Lerna's distinctive Cardano/Hydra contribution is to move the ordinary case into Hydra snapshots and reserve heavyweight adjudication for the future V1 claim-UTxO mode.

## 4. System Model

### Cardano L1

Cardano L1 is the settlement layer. It supplies eUTxO accounting, native assets, Plutus/Aiken validators, reference inputs, inline datums, reference scripts, transaction validity intervals, and min-ADA requirements. Lerna assumes no Cardano consensus change.

### Hydra Head

A Hydra Head is the outer L2 execution environment. Participants submit transactions to Hydra nodes; the Head state evolves through signed snapshots. Any participant can close with a snapshot; others may contest with newer valid snapshots before the contestation deadline; final fanout distributes the resolved Head UTxO set back to L1. Lerna V0 assumes an open Head and uses only this existing machinery.

### Lerna Factory

A Lerna factory is a compact Head-local object representing many virtual channels and their rights. In V0 it is a Head-internal UTxO or state object controlled by a validator or by agreed Head-local transaction rules. It contains commitments to balances, subchannels, membership, reserve claims, release receipts, and materialisation descriptors.

### Virtual Channel Participants

Virtual-channel participants may be the same as Head participants, a subset of them, or users represented by Head participants. The distinction matters. If a virtual participant is not a Head participant, the design must state who can submit, observe, contest, or materialise on that participant's behalf. Lerna V0 therefore treats operator and watcher responsibilities as part of the deployment profile.

### Watchers And Operators

Hydra already requires participants to remain sufficiently online for safe contestation. Lerna adds inner obligations: tracking latest virtual-channel states, checking release receipts, detecting stale local exits, and preparing materialisation plans before Head closure. Operators may pay ordinary Head-level transaction fees or manage application liquidity. They do not receive implicit authority over factory-owned balances.

## 5. Threat Model And V0 Scope

V0 has a sharp boundary: it does not give a non-Head virtual participant an independent L1 exit path. The enforceable outer object is the Hydra snapshot that survives close and contest.

Actors:

- Head participant: can run a Hydra node, sign snapshots, submit Head-local transactions, close the Head, and contest with a newer snapshot.
- Virtual participant: owns a right inside the Lerna factory; may or may not be a Head participant.
- Operator: may batch updates, fund ordinary fees, or coordinate materialisation, but has no implicit authority over balances.
- Watcher: observes Lerna and Hydra evidence; in V0 it matters only if it has a live path to an authorised Head participant.

The adversary may corrupt operators, watchers, virtual participants, and any subset of Head participants. It may withhold signatures or proofs, propose same-number conflicting states, censor checkpoint inclusion, close with a stale snapshot, delay materialisation, or create too many compact rights to unfold safely.

V0 does not assume an honest majority. It requires at least one non-corrupt authorised actor with the evidence and authority path needed to checkpoint or materialise the newer Lerna state into a Hydra snapshot before the contestation deadline. If all such Head-side actors censor or fail, V0 fails; the remedy is V1 claim-UTxO research or a different custody model, not stronger wording.

## 6. Design Thesis

Hydra provides fast isomorphic eUTxO execution. Lerna provides virtual-channel factory discipline inside Hydra-family Heads.

The thesis has five parts.

1. Compactness: many virtual channels can be represented by one factory state object inside a Head, avoiding Head-internal UTxO explosion during normal operation.

2. Deterministic unfoldability: compact state must always admit a deterministic materialisation plan into ordinary Head UTxOs, because Hydra fanout ultimately distributes UTxOs rather than abstract rights.

3. Latest-state settlement: each virtual channel and factory transition uses monotonic state numbers and domain-separated signatures. Newer valid Lerna states supersede older Lerna states at the factory layer, without penalties.

4. Non-interference: reduced signing sets are safe only when touched leaves and dependency proofs establish that untouched participants' rights are unchanged.

5. V0 before V1: the first practical implementation should keep all Lerna enforcement inside an open Hydra Head. L1 claim UTxOs are future research unless and until validators can enforce the necessary semantics within Cardano limits.

## 7. Formal Objects

This section defines protocol objects at the design level. Encoding, CBOR schema, validator budget, and Aiken/Plutus implementation details are deliberately left for later specifications.

### LernaFactoryConfig

```text
LernaFactoryConfig {
  protocol_version
  network_id
  hydra_head_id
  factory_id
  factory_epoch
  head_participant_set_hash
  virtual_participant_registry_root
  ledger_profile              // V0-ADA, multi-asset-extension, V1-claim
  asset_registry_root
  reserve_policy_hash
  fee_budget_policy_hash
  materialisation_policy_hash
  signature_scheme_id
  domain_separator
}
```

The configuration binds the factory to a Cardano network, a Head, a factory identity, a participant registry, and a materialisation policy. `factory_epoch` changes when the compact factory value is re-anchored, for example after a major splice, reserve restructure, or validator upgrade. The concrete V0 target sets `ledger_profile = V0-ADA` and rejects non-ADA assets.

### FactoryState

```text
FactoryState {
  config_hash
  factory_state_number
  balances_root
  subchannels_root
  membership_root
  reserve_root
  release_receipt_root
  pending_materialisation_root
  settlement_descriptor_hash
  asset_registry_hash
  operator_budget_state
}
```

`FactoryState` is the compact state of the Lerna factory. It is not a Cardano transaction by itself. It is the object that Head-local transactions update and that materialisation plans unfold.

### VirtualChannelState

```text
VirtualChannelState {
  virtual_channel_id
  factory_id
  participants
  virtual_state_number
  balances
  locked_reserve
  app_state_hash
  settlement_descriptor_hash
  expiry_or_liveness_profile
}
```

`VirtualChannelState` is the latest-state object for one virtual channel. In a payment-only prototype, `app_state_hash` may be empty and balances are enough. In a dApp channel, `app_state_hash` commits to application-specific eUTxO or state-machine material.

### LatestStateHeader

```text
LatestStateHeader {
  domain_separator
  protocol_version
  network_id
  hydra_head_id
  factory_id
  factory_epoch
  state_scope              // factory, virtual-channel, local-exit, splice
  state_number
  previous_state_hash
  new_state_hash
  transition_family
  touched_root
  settlement_descriptor_hash
  materialisation_plan_hash
  validity_interval
  hydra_snapshot_hint
}
```

The header is the signed authority object. It must be domain-separated from ordinary Cardano transactions, Hydra snapshots, wallet messages, and earlier protocol versions. `hydra_snapshot_hint` is not authority by itself; it helps watchers correlate Lerna states with Head progress.

V0 signatures are Ed25519 over canonical CBOR bytes, with a Blake2b-256 digest and explicit domain tags:

- `lerna:v0:latest-state-header`
- `lerna:v0:release-receipt`
- `lerna:v0:materialisation-plan`
- `lerna:v0:watcher-delegation`

The signer registry must say whether a key is a Hydra party key, Cardano payment/stake key used for off-chain messages, or application key. These roles are not interchangeable.

### AccessManifest

```text
AccessManifest {
  transition_family
  authorisation_policy_id
  affected_participants
  required_signers
  read_set_root
  write_set_root
  inserted_set_root
  removed_set_root
  dependency_set_root
  asset_delta_summary
  reserve_delta_summary
}
```

The access manifest is the envelope-first admission object. It declares what kind of transition is being attempted before proof bytes are interpreted.

### TouchedLeaves

```text
TouchedLeaves {
  balance_leaves
  subchannel_leaves
  membership_leaves
  reserve_leaves
  release_receipt_leaves
  absence_proofs
  membership_proofs
}
```

Touched leaves are the concrete items read, written, inserted, removed, or proven absent. They must be sufficient to recompute the old and new roots under the declared transition family.

### NonInterferenceProof

```text
NonInterferenceProof {
  access_manifest_hash
  rights_dependency_graph_hash
  untouched_rights_commitment
  unchanged_dependency_paths
  conservation_witness
}
```

The proof must establish that all rights outside the transition footprint remain unchanged, including indirect rights through shared reserve, membership, exit paths, and sponsor or operator budgets. A Merkle proof alone is not a non-interference proof.

### SettlementDescriptor

```text
SettlementDescriptor {
  descriptor_version
  output_derivation_rules
  participant_destination_policies
  asset_allocation_rules
  min_ada_rules
  datum_and_reference_script_rules
  local_exit_rules
  rebalance_and_splice_rules
  fanout_preparation_rules
}
```

The descriptor is not an application runtime. It is a bounded rule set for deriving Cardano outputs or Head-local outputs from Lerna rights.

### FanoutMaterialisationPlan

```text
FanoutMaterialisationPlan {
  plan_id
  source_factory_state_hash
  materialisation_mode      // in-head, zero-release, claim-utxo
  output_set_hash
  claim_utxo_descriptor_hash
  unresolved_channel_set_root
  batch_index
  batch_count
  budget_profile_hash
  required_signatures
  watcher_deadline_profile
}
```

In V0, `claim_utxo_descriptor_hash` must be empty because unresolved factory state is not allowed to survive Head fanout. In V1 it may describe a script-controlled L1 claim UTxO.

### ReleaseReceipt

```text
ReleaseReceipt {
  receipt_id
  receipt_domain_separator
  released_right_id
  factory_id
  factory_epoch
  virtual_channel_id
  state_number
  right_commitment_hash
  materialisation_plan_hash
  batch_index
  output_index
  replacement_output_hash
  consumed_in_factory_state_hash
  signer_set_hash
  signature
}
```

Release receipts are a Hydra-native addition. They prevent a common ambiguity in compact factories: after a virtual right is materialised into an ordinary Head UTxO, the compact factory must no longer carry a spendable copy of that right.

Receipt anti-replay rule:

```text
receipt_id =
  H("lerna:v0:release-receipt",
    network_id,
    hydra_head_id,
    factory_id,
    factory_epoch,
    released_right_id,
    state_number,
    materialisation_plan_hash,
    batch_index,
    output_index)
```

A receipt is valid only if this identifier is absent from the old `release_receipt_root`, present in the new `release_receipt_root`, the released right is absent from the new active-right commitments, and the replacement output hashes exactly to `replacement_output_hash`.

## 8. Minimal ADA-only V0 State Machine

The V0 prototype should be specified as a small ADA-only state machine before any multi-asset or V1 claim machinery is attempted.

Factory mode:

```text
Init
Open
Checkpointing
Materialising
ShutdownPreparing
Settled
Aborted
```

State variables:

```text
F = (config_hash,
     factory_state_number,
     mode,
     balance_rights,
     reserve_rights,
     pending_materialisation_root,
     release_receipt_root,
     participant_registry_root,
     settlement_descriptor_hash,
     operator_budget_state)
```

Core transitions:

| Transition | Mode change | Admissibility summary |
|---|---|---|
| `initFactory` | `Init -> Open` | Head id, factory id, descriptor, signer registry, and ADA-only ledger profile are bound. |
| `openVC` | `Open -> Open` | Required signers authorise a virtual right; reserve and min-ADA accounting increase consistently; non-ADA assets are rejected. |
| `checkpoint` | `Open -> Checkpointing -> Open` | A domain-separated latest-state header strictly increases the relevant state number and is included in a Hydra snapshot. |
| `rebalance` | `Open -> Open` | Source and destination rights are declared in the manifest; ADA is conserved across affected rights and reserve lanes. |
| `localExit` | `Open -> Materialising` | Latest evidence, touched leaves, and either full factory authorisation or active non-interference proof authorise selected materialisation. |
| `materialiseBatch` | `Materialising -> Materialising/Open` | Canonical batch creates ordinary Head outputs, writes fresh receipts, removes compact rights, and updates the pending suffix commitment. |
| `prepareShutdown` | `Open -> ShutdownPreparing` | Materialisation debt exceeds the configured threshold or the Head is expected to close; new compact opens stop. |
| `cooperativeSettle` | `Open/ShutdownPreparing -> Settled` | All remaining rights are materialised, released, or covered by full-participant shutdown signatures. |
| `abortV0` | any non-settled mode -> `Aborted` | Analytical state when liveness bridge or materialisation capacity is violated; V0 has no independent recovery from it. |

`ValidV0ADA(F, F', T, W)` holds only when factory identity, Head binding, strict state-number monotonicity, signature-domain correctness, manifest-to-body binding, touched-root recomputation, release-receipt freshness, ADA conservation, min-ADA reserve sufficiency, and operator-budget separation all hold. If fewer than all factory signers authorise a transition, `ValidNonInterference` must be active and benchmarked; otherwise the transition is invalid.

Same-number equivocation is not ordered. For a fixed `(network, head, factory, epoch, scope)`, two different headers with the same `state_number` and different `new_state_hash` freeze the affected scope until a strictly higher repair state or full shutdown is signed.

## 9. Protocol Lifecycle

### 9.1 Open Head

Participants open a Hydra Head under the ordinary Hydra lifecycle. In recent Hydra design, an Init transaction can directly open an empty Head and funds are added through deposit/increment. Lerna does not modify this process.

### 9.2 Initialise Lerna Factory

The Head participants create an initial Lerna factory state inside the open Head. A conservative prototype may represent it as one script-controlled Head UTxO with an inline datum containing the `FactoryState` hash and enough data to update it. More compact implementations may store only the roots and keep proofs off-chain.

Initialisation must bind:

- the Cardano network id;
- the Hydra Head id;
- the Lerna factory id;
- the participant registry root;
- the asset registry root;
- the settlement descriptor hash;
- the materialisation policy.

### 9.3 Open Virtual Channel

Opening a virtual channel updates the factory state. The transition touches membership, reserve, balances, and subchannel roots. It requires signatures from the virtual-channel participants and any factory participants whose reserve or exit rights are affected. If the opening changes shared reserve or membership in a way that affects everyone, full factory authorisation is required unless non-interference proves otherwise.

### 9.4 Off-chain Or Head-local Update

There are two update styles.

The first style is purely off-chain between the virtual-channel participants. They exchange signed `LatestStateHeader` objects and only submit to the Head when they need to checkpoint, rebalance, exit, or respond to conflict.

The second style is Head-local. A compact factory UTxO is updated by a transaction inside the Head, and the resulting Head snapshot records the new factory roots. This costs Head-level throughput but gives stronger shared observability.

The V0 prototype should support Head-local checkpointing first, then add purely off-chain virtual updates once materialisation and release receipts are well tested.

### 9.5 Local Rebalance Or Splice

A rebalance moves value between virtual channels without changing the outer Head deposit. A splice changes the compact factory reserve by depositing into, decommitting from, or rearranging Head-local value. Lerna must distinguish these carefully:

- rebalance changes rights inside the factory;
- splice changes the factory's Head-visible value or materialisation reserve;
- neither may silently change the factory id or replay domain.

### 9.6 Local Exit

A local exit lets a participant materialise a virtual-channel right without closing the entire factory. It requires:

- latest-state evidence for the relevant virtual channel;
- release receipts for rights removed from compact state;
- touched leaves and membership proofs;
- non-interference proof for untouched participants, or full factory authorisation;
- exact output materialisation under the settlement descriptor.

### 9.7 Materialise Inside Head

Materialisation converts compact factory rights into ordinary Head UTxOs. In V0 this is the main safety operation. Before Head close, every unresolved virtual channel must either:

- materialise into ordinary Head outputs;
- be reabsorbed into the factory with fresh signatures;
- be closed with release receipts;
- be covered by a full-participant shutdown plan.

The materialised outputs must satisfy Cardano eUTxO constraints, especially min-ADA. In V0-ADA, non-ADA assets in the factory input or materialised outputs are rejected rather than handled.

Deterministic batch rule:

1. Sort rights by `(factory_id, virtual_channel_id, right_kind, participant_id, nonce)`.
2. Derive candidate outputs from the settlement descriptor.
3. Reject any output that fails output validity or min-ADA requirements.
4. Compute `Fits_P(k)` under the committed `BudgetProfile`, including output bytes, datum updates, receipts, and fixed witness-size estimate.
5. Materialise exactly `k* = max { k | 0 < k <= |R| and Fits_P(k) }`.
6. Commit the remaining suffix in `pending_materialisation_root`, increment `batch_index`, and write fresh release receipts for the prefix.

Only a confirmed Head-local transaction changes the factory state. If a batch fails before confirmation, the last confirmed state remains authoritative and the same batch can be reconstructed. If earlier batches confirmed but the Head closes before the suffix is empty, V0 is safe only if a newer Hydra snapshot containing further materialisation can still be contested or if the remaining rights are covered by full-participant shutdown signatures.

### 9.8 Head Close, Contest, And Fanout

If the Head is closed, Hydra's own close and contest rules decide which Head snapshot is final. Lerna V0 does not create a separate L1 dispute. Therefore, Lerna participants must ensure the final Head snapshot contains either ordinary materialised outputs or a fully agreed factory shutdown state that has no unresolved unilateral claims.

If a stale Head snapshot omits a newer Lerna materialisation, the answer is not a Lerna-specific L1 action. The answer is to contest the Head with a newer Hydra snapshot that includes the correct Lerna state or materialised outputs. This is why watcher responsibilities must cover both Hydra snapshots and Lerna latest-state evidence.

### 9.9 V1 Claim-UTxO Fanout

V1 is future work. If the Head closes before all virtual channels are materialised, the final Head UTxO set may include a script-controlled L1 claim UTxO representing the unresolved factory state. An Aiken or Plutus validator would then enforce:

- domain-separated latest-state signatures;
- state monotonicity;
- local exits;
- non-interference proofs;
- native-asset conservation;
- min-ADA-safe output creation;
- release receipt consumption.

This is research-heavy. Until such validators are specified and tested, V1 should not be presented as deployable.

## 10. Latest-State Settlement Semantics

Lerna uses monotonic state numbers. For a fixed `(network_id, hydra_head_id, factory_id, factory_epoch, state_scope)`, a valid state with number `n + k` supersedes a valid state with number `n` at the Lerna layer if:

- the header is domain-separated;
- signatures match the required signing set;
- the transition family is admissible;
- touched leaves recompute the old and new roots;
- conservation checks pass;
- release receipts prevent double materialisation.

There is no penalty mechanism. A participant presenting stale evidence does not lose funds by punishment. The safety objective is simply that stale evidence cannot finalise if newer valid evidence is available through the appropriate Hydra contest or Lerna materialisation path.

V0 only has a Hydra inclusion lemma: if an honest authorised actor observes a newer valid Lerna state, can construct a checkpoint or materialisation transaction, obtains a newer Hydra snapshot containing it, and contests before the deadline, then Hydra fanout follows the newer snapshot. This is not a standalone Lerna security theorem. The hard premises are authorised inclusion, proof availability, materialisation capacity, receipt consumption, and timely contest.

In V1, a claim validator may enforce latest-state settlement on L1, but this paper treats that as future work.

## 11. Factory-local Proof Mode

A Lerna factory may contain many balances and subchannels. Requiring all participants to sign every local action is safe but undermines the purpose of a factory. Requiring only affected participants to sign is unsafe unless the protocol proves that untouched participants are not affected.

The proof mode has four layers.

1. Commitment roots: balances, subchannels, membership, reserve, and release receipts are committed separately.

2. Access manifest: the transition family and footprint are declared before proof interpretation.

3. Touched leaves: the transition supplies the leaves it reads, writes, inserts, removes, or proves absent.

4. Non-interference proof: the transition proves that untouched rights remain unchanged, including dependency paths through shared reserve and membership.

Rights dependency graph:

```text
G_F = (V, E, labels)
V = balance rights
  ∪ reserve rights
  ∪ membership rights
  ∪ descriptor rights
  ∪ receipt rights
  ∪ operator-budget rights
```

An edge `u -> v` means changing `u` may change the value, exit path, materialisation viability, or authorisation condition of `v`. For a manifest `M`, the touched closure is the declared read/write/insert/remove/absence set plus every graph successor reachable from it. Reduced authorisation is admissible only if every right outside that closure has unchanged membership, balance, reserve, descriptor, and exit-path commitments.

If non-interference cannot be proven, Lerna falls back to full factory authorisation. This is not a weakness; it is the correct conservative boundary. Locality is a verification property, not a social wish.

## 12. Conservation Model

The first V0 ledger profile is ADA-only. Native assets are an extension target because Hydra's known limitations around unburned tokens and large output bundles make native-asset fanout an unsafe first claim.

For a factory state `F`, define:

- `B(F)`: spendable lovelace owed by active virtual rights.
- `M(F)`: lovelace reserved only for min-ADA of materialised outputs.
- `R(F)`: shared factory reserve not assigned to one virtual balance.
- `O(F)`: operator budget authorised for fees and publication costs.
- `P(F)`: pending materialised-output lovelace committed by the current plan.

For an ordinary internal update with no Head-visible splice and no materialisation:

```text
B(F) + M(F) + R(F) + O(F) + P(F)
=
B(F') + M(F') + R(F') + O(F') + P(F')
```

For a materialisation batch producing ordinary Head outputs `Y` and explicitly consuming operator-budget amount `fee_O`:

```text
B(F) + M(F) + R(F) + O(F) + P(F)
=
B(F') + M(F') + R(F') + O(F') + P(F') + ada(Y) + fee_O
```

Every materialised output must satisfy the settlement descriptor and min-ADA requirement. Min-ADA reserve is not spendable business balance. A local exit that pays by stealing from `M(F)`, `R(F)`, or `O(F)` is invalid unless the transition family and signer set explicitly authorise that lane movement.

Future multi-asset profiles must add one conservation equation per `(policy_id, asset_name)`. Metadata, ticker symbols, and display names are not asset identity. This is not part of the ADA-only V0 prototype claim.

## 13. Hydra L2/L3 Interaction

Lerna is naturally an inner layer.

At L1, Cardano provides settlement, validators, native assets, and finality.

At L2, Hydra or a Hydrozoa-style Head provides fast shared execution and snapshot/fanout safety.

Inside the Head, Lerna provides virtual-channel factory semantics.

Above Lerna, applications can run payment channels, app-specific channels, local rebalances, and conditional exits.

This can be described as L3 only if the term is used carefully. Lerna is not a new global consensus layer above Hydra. It is a factory discipline whose ordinary state transitions can be off-chain relative to the Head and whose checkpoints can be Head-local.

Compatibility with Interhead and virtual Head directions is plausible but not solved. A virtual channel could connect liquidity positions represented in two Heads, but then Lerna needs cross-Head evidence, timeout alignment, and liquidity-discovery logic. Those are out of scope.

Hydrozoa dynamic membership may make Lerna more useful, not less. If Head membership can evolve, the factory participant registry and rights-dependency graph become even more important. Lerna should not assume static Head membership forever, but V0 should begin with a fixed participant set.

## 14. Security Properties

This section states target safety properties and the mechanism that is expected to enforce each one.

### Anti-replay

State signatures cannot be replayed across networks, Heads, factories, factory epochs, state scopes, or descriptor versions because `LatestStateHeader` binds those fields under a domain separator.

Assumptions: signature unforgeability; canonical encoding; no domain-separator collision.

### State Monotonicity

For a given state scope, valid transitions strictly increase `state_number`. V0 enforces this through Head-local factory update rules and by requiring any contested Head snapshot to include the latest valid Lerna checkpoint or materialisation.

Assumptions: watchers can observe relevant Head snapshots and submit contesting snapshots within Hydra's contestation period.

### Virtual-channel Balance Conservation

In V0-ADA, lovelace balances are conserved per virtual channel unless a transition explicitly moves value through a declared rebalance, splice, deposit, exit, or release.

Mechanism: access manifest, touched leaves, reserve delta checks, settlement descriptor, and the ADA equations above.

### Factory Non-interference

Untouched participants' rights remain unchanged when a reduced signing set authorises a local action.

Mechanism: rights-dependency graph, non-interference proof, and fallback to full authorisation when proof is insufficient.

### Safe Materialisation

Compact rights unfold into valid Cardano outputs exactly once.

Mechanism: settlement descriptor, min-ADA rules, materialisation plan, and release receipts.

### Close/Fanout Safety

In V0, no unresolved unilateral virtual claim should survive Head fanout.

Mechanism: materialisation-before-fanout rule and Hydra contestation with newer snapshots when stale Head snapshots omit required Lerna state.

### No Accidental Head-level UTxO Explosion

The factory normally remains compact inside the Head.

Mechanism: compact `FactoryState` plus selective materialisation. The protocol deliberately postpones UTxO expansion until local exit, rebalance boundary, or Head shutdown preparation.

### Native-asset Extension No-drift Target

In a future multi-asset profile, native assets must not appear or disappear merely because a compact proof changes a root.

Proposed mechanism: per-policy asset conservation, explicit mint/burn policy checks when relevant, and materialised-output equality under the descriptor. This is an extension target, not part of the ADA-only V0 claim.

## 15. Failure Scenarios

| Scenario | What happens in V0 | Design consequence |
|---|---|---|
| Operator refuses to checkpoint | A non-Head virtual participant cannot force inclusion through Lerna alone. | Deployment must name an authorised fallback actor or admit assisted, not unilateral, security. |
| Head participant censors materialisation | If no honest authorised participant can create a newer snapshot, stale Head state may finalise. | V0 fails; V1 claim-UTxO or a different custody model is required. |
| Watcher is late | Newer Lerna state may be valid but irrelevant after the contest deadline. | Watcher response time is part of the materialisation threshold. |
| Proof material withheld | A signed high-number header without leaves, descriptor body, or receipt witnesses cannot be validated. | Freeze and require repair or full authorisation unless proof availability was bound at signing. |
| Same-number equivocation | Same scope and number with different state hashes has no ordering relation. | Freeze until a strictly higher repair state or full shutdown is signed. |
| Too many outputs | Deterministic materialisation cannot complete with margin. | Stop compact opens and enter shutdown preparation before close risk. |
| Receipt replay attempt | A right appears as both compact state and materialised output. | Reject because receipt ids are fresh, output-bound nullifiers. |
| Native-asset bundle appears in V0-ADA | Factory input or materialised output contains non-ADA assets. | Reject under `ledger_profile = V0-ADA`; multi-asset needs a separate profile. |

## 16. Limitations And Open Questions

Hydra fanout constraints are central. A Lerna plan that materialises too many outputs too late may fail in practice even if it is theoretically valid. Partial fanout helps but does not remove intrinsic Cardano transaction-size limits, especially for large datums or awkward native-asset bundles.

Head participant limits also matter. Lerna can reduce virtual-channel signing sets, but the outer Head still has a participant set and networking topology. Non-Head users need representation and watcher arrangements.

V1 claim UTxOs are complex. A validator must handle signatures, monotonicity, local exits, non-interference, asset conservation, min-ADA, and proof sizes within Cardano execution budgets. This paper does not claim that such a validator is complete.

Proof size is unknown. Sparse Merkle proofs, KZG-style accumulators, or other commitments may be useful, but the choice must be benchmarked against Aiken/Plutus costs and Head-local transaction ergonomics.

Routing and liquidity discovery are out of scope. Lerna defines factory-local safety, not a payment network.

Privacy is mostly out of scope. Commitment roots avoid full plaintext disclosure in factory mode, but access manifests and touched leaves still leak information.

Production readiness is not claimed. The immediate output should be a prototype and a test agenda.

## 17. Evaluation Agenda

The first evaluation should be small and concrete.

### Minimal Hydra Local Demo

Run a local Hydra Head with two or three participants. Create a compact Lerna factory UTxO inside the Head. Execute virtual-channel open, update, local exit, and materialisation transactions. Close and fan out only after materialisation.

### Minimum Prototype Measurements

The first report should include these ADA-only measurements rather than generic evaluation prose:

| Measurement | Prototype case | Why it matters |
|---|---|---|
| Head-local transaction bytes | open, checkpoint, local exit, materialise batch | Determines the safe batch budget. |
| Validator execution units | compact factory spend | Shows whether Aiken/Plutus checks are plausible. |
| Materialised output count per batch | ADA-only exits | Bounds shutdown preparation. |
| Inline datum bytes | compact factory datum and materialised outputs | Detects unfannable large-output risk. |
| Min-ADA locked for materialisation | two-channel local exit | Separates business balance from output viability reserve. |
| Receipt bytes and root growth | repeated local exits | Shows whether receipts become datum bloat. |
| Watcher response time | stale close then contest | Tests the liveness bridge. |

### Aiken Validator Sketch

Prototype an Aiken validator for the Head-local factory UTxO. It should check domain separation, state-number monotonicity, touched-leaf root updates, release receipts, and a simple ADA-only conservation rule.

### Virtual Channel Update Traces

Generate traces for:

- cooperative update;
- stale local exit attempt;
- newer-state materialisation;
- rebalance between two virtual channels;
- release receipt creation;
- failed non-interference proof.

### Close/Fanout Test Cases

Test Head closure with:

- all virtual channels materialised;
- stale Head snapshot missing materialisation, then contest with newer snapshot;
- excessive materialisation output count;
- large inline datum;
- rejection of native-token outputs under `ledger_profile = V0-ADA`.

### Non-interference Positive/Negative Tests

Positive tests should show that a local exit touching Alice and Bob does not affect Carol's balance, reserve claim, membership, or exit path. Negative tests should show that a balance-only proof is rejected when shared reserve changes.

### Extension Asset Conservation Matrix

The V0 acceptance path tests lovelace and min-ADA only. The extension suite should later test one native asset, multiple native assets under the same policy, multiple policy IDs, zero-quantity edge cases, and explicit mint/burn boundaries.

## 18. Conclusion

Lerna Protocol is best understood as a Hydra-native factory discipline. It does not try to replace Hydra's Head lifecycle, does not compete with Hydrozoa, and does not rebrand eltoo for Cardano. Its first credible purpose is narrower: test whether an ADA-only compact factory inside a Hydra Head can preserve deterministic materialisation, receipt-backed anti-replay, and explicit conservation without creating unsafe fanout debt.

The most promising first implementation is V0-ADA. Keep Lerna entirely inside an open Hydra Head. Use a compact factory UTxO. Require unresolved virtual rights to materialise before Head close and fanout. Add release receipts so compact rights cannot be double-spent after materialisation. Use full factory authorisation unless non-interference is implemented and benchmarked. Treat native assets and V1 L1 claim UTxOs as future research until validators, proof sizes, and fanout behaviour are credible.

If this direction is correct, Lerna fills a real gap in the Cardano scaling stack: Hydra gives fast shared eUTxO execution; Hydrozoa and Interhead work broaden the Head-family design space; Lerna supplies the factory-local semantics needed to make virtual channels safe, compact, and auditable inside that ecosystem.

## References

- Hydra repository and documentation: https://github.com/cardano-scaling/hydra and https://hydra.family/head-protocol/docs/protocol-overview
- Hydra Head paper, "Fast Isomorphic State Channels": https://eprint.iacr.org/2020/299.pdf
- Hydra known issues and limitations: https://hydra.family/head-protocol/docs/known-issues
- Hydrozoa repository and specification notes: https://github.com/cardano-hydrozoa/hydrozoa
- Aiken repository and documentation entry point: https://github.com/aiken-lang/aiken and https://aiken-lang.org/
- Cardano eUTxO documentation: https://docs.cardano.org/about-cardano/learn/eutxo-explainer/
- eltoo paper: https://blockstream.com/eltoo.pdf
- Decker, Russell, Osuntokun, "eltoo: A Simple Layer2 Protocol for Bitcoin": https://blockstream.com/eltoo.pdf
- Burchert, Decker, Wattenhofer, "Scalable Funding of Bitcoin Micropayment Channel Networks": https://tik-old.ee.ethz.ch/file//49218d6b9325ddb4f18aef8830ec567b/1/scalable_funding.pdf
