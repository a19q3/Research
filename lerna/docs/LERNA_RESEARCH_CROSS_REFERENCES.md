# Lerna Research Cross References

Status: source map and relevance notes  
Date: 2026-06-05

## Local Material Inspected

### Morph Channel

Path: `/Users/arthur/RustroverProjects/Research-a19q3/morph_channel/paper.tex`

Relevant ideas:

- stable funding identity and moving state evidence;
- state-header-centric signing;
- sponsor-owned publication boundary;
- partition conservation;
- factory proof mode;
- access manifests and touched leaves;
- non-interference for untouched participants;
- exact materialisation;
- conservative language around production readiness.

Lerna use:

- Preserve the conceptual separation of value, state evidence, fee budget, and factory proof authority.
- Replace CKB Cells, type scripts, lock scripts, xUDT, and `since` with Cardano/Hydra-native objects.
- Keep Morph's conservative proof standards.

### Hydra

Path: `/Users/arthur/RustroverProjects/hydra`

Relevant local files:

- `README.md`: implementation of Hydra scalability protocols and Hydra Head.
- `docs/docs/protocol-overview.md`: Head lifecycle, snapshots, close, contest, fanout, deposit, decommit.
- `docs/docs/known-issues.md`: participant limits, transaction size limits, native-token fanout issue, static topology, deposit constraints.
- `docs/docs/how-to/incremental-commit.md`: deposit mechanics, blueprint transactions, min-ADA, reference inputs.
- `docs/docs/how-to/incremental-decommit.md`: decommit mechanics.
- `docs/adr/2026-03-10_033-directly-open-head.md`: removal of the initialisation phase and direct opening of empty Heads.
- `hydra-tx/src/Hydra/Tx/Snapshot.hs`: snapshot number, version, UTxO hash, accumulator, multisignature representation.
- `hydra-tx/src/Hydra/Tx/Close.hs`: close transaction and contestation deadline.
- `hydra-tx/src/Hydra/Tx/Contest.hs`: contest transaction and deadline extension logic.
- `hydra-tx/src/Hydra/Tx/Fanout.hs`: full fanout, partial fanout, final partial fanout.

Lerna use:

- V0 must be an application-layer protocol inside an open Head.
- Hydra snapshots are the outer latest-state evidence.
- Head close/contest/fanout must remain authoritative.
- Materialisation must avoid creating UTxOs that Hydra cannot fan out.
- Deposit/decommit and blueprint mechanics suggest how Lerna can integrate with dApps.

### Hydra Pay

Path: `/Users/arthur/RustroverProjects/hydra-pay`

Relevant local files:

- `README.md`: payment-channel API, automatic Head lifecycle management, internal wallets, WebSocket API, standardisation concerns, decommit integration as a next step.

Lerna use:

- Shows a community-facing layer that hides raw Hydra complexity from builders.
- Suggests that Lerna should expose factory and virtual-channel operations as application objects, while still respecting Hydra lifecycle.
- Standardisation of names, participant identifiers, and internal wallet roles is a practical issue for any Lerna operator.

### Hydrozoa

Path: `/Users/arthur/RustroverProjects/hydrozoa`

Relevant local files:

- `README.md`: pre-alpha status; execution/custody decoupling; continuous deposits and withdrawals; rule-based fallback; Cardano-first multi-L1 direction; old spec deprecated and new whitepaper WIP.
- `specification/frontmatter/04-overview.tex`: dynamic near-isomorphic multi-party state channel; multisig and rule-based regimes; deposits and withdrawals; L2 ledger state and blocks.
- `TOKEN_RECOVERY.md`: temporary token recovery concerns around ADA, tokens, head tokens, and min-ADA.

Lerna use:

- Hydrozoa is not a competitor in this paper.
- Its execution/custody separation supports Lerna's view that factory state and custody/fanout need not be conflated.
- Its rule-based fallback motivates V1 claim-UTxO research, but the current public status means Lerna should not rely on a final Hydrozoa API.

### Aiken

Path: `/Users/arthur/RustroverProjects/aiken`

Relevant local files:

- `README.md`: Aiken as a modern smart contract platform for Cardano, with documentation and examples.
- `examples/hello_world/README.md` and `examples/gift_card/README.md`: basic validator orientation.

Lerna use:

- Aiken is a plausible implementation language for V0 factory validators and V1 claim validators.
- The paper should avoid promising validator feasibility until proof size and execution budget are measured.

## Public References To Cite

### Hydra Documentation

URL: https://hydra.family/head-protocol/docs/protocol-overview

Use:

- Head lifecycle.
- Open Head, snapshots, close, contest, fanout.
- Isomorphic eUTxO execution.

### Hydra Known Issues

URL: https://hydra.family/head-protocol/docs/known-issues

Use:

- Participant and transaction-size constraints.
- Native-token fanout hazards.
- Static topology and operational limitations.

### Hydra Head Paper

URL: https://eprint.iacr.org/2020/299.pdf

Use:

- Fast isomorphic state channels.
- Academic grounding for Hydra Head.

### Hydrozoa Repository

URL: https://github.com/cardano-hydrozoa/hydrozoa

Use:

- Current public project positioning.
- Pre-alpha and active development status.
- Execution/custody decoupling and rule-based fallback.

### Aiken

URLs:

- https://github.com/aiken-lang/aiken
- https://aiken-lang.org/

Use:

- Aiken as a Cardano smart-contract implementation path.
- Validator sketches and future experiments.

### Cardano eUTxO

URL: https://docs.cardano.org/about-cardano/learn/eutxo-explainer/

Use:

- Cardano eUTxO terminology.
- Basis for explaining UTxO-level settlement and script-controlled outputs.

### eltoo

URL: https://blockstream.com/eltoo.pdf

Use:

- Non-punitive latest-state lineage.
- Only background, not positioning.

### Channel Factories

URL: https://tik-old.ee.ethz.ch/file//49218d6b9325ddb4f18aef8830ec567b/1/scalable_funding.pdf

Use:

- Shared funding and factory motivation.

## Claims To Be Careful With

Do not claim:

- Lerna is production-ready.
- Lerna replaces Hydra.
- Lerna is eltoo on Cardano.
- Lerna competes with Hydrozoa.
- Lerna V1 is solved.
- Lerna V0 gives non-Head virtual participants an independent L1 exit path.
- Lerna V0 proves reduced-signature non-interference.
- Lerna V0 supports native assets safely.
- Non-interference follows from Merkle inclusion alone.
- A compact factory state can always be fanned out safely.
- Native tokens are safe merely because Cardano supports native assets.

Do claim, with qualifications:

- Lerna V0 can be a practical POC because it stays inside Hydra.
- Lerna V0 should be ADA-only until materialisation capacity and receipt logic are measured.
- Lerna adds a distinct inner factory discipline.
- Materialisation-before-fanout is the key conservative safety rule.
- Release receipts are a useful Hydra-native addition.
- V1 claim UTxOs are a research direction requiring validator and proof-size work.

## Community Positioning Notes

Cardano developers tend to value correctness around eUTxO accounting, explicit validator assumptions, and honest treatment of limitations. The paper should therefore sound like a design note for builders, not a token whitepaper.

Use Cardano-native vocabulary:

- Hydra Head;
- Head participant;
- snapshot;
- contestation period;
- close;
- contest;
- fanout;
- deposit;
- decommit;
- eUTxO;
- native assets;
- lovelace;
- min-ADA;
- policy ID;
- asset name;
- inline datum;
- reference input;
- reference script;
- Plutus;
- Aiken.

Avoid imported vocabulary unless it is being mapped:

- CKB Cell;
- State Cell;
- vault Cell;
- xUDT;
- `since`;
- eltoo-style replacement transaction.

## Reference-backed Design Constraints

1. Hydra fanout distributes UTxOs, so Lerna compact rights must unfold into UTxOs or become a carefully designed claim UTxO.

2. Hydra close and contest already define the outer dispute mechanism, so Lerna V0 must not add a separate L1 dispute route.

3. Hydra known issues around large UTxOs and native-token fanout make materialisation planning part of safety, not just performance.

4. Cardano native assets require identity by policy ID and asset name. Token names or metadata are not sufficient.

5. min-ADA is not business balance. It is output viability reserve and must be carried explicitly in settlement descriptors.

6. Hydrozoa's execution/custody decoupling is an important compatibility signal, but the public project status is pre-alpha/WIP, so Lerna should not depend on a stable Hydrozoa interface.

7. Aiken/Plutus validator feasibility must be measured. Until then, V1 remains future work.

8. For V0, the honest actor assumption is not an honest majority claim. The relevant condition is that at least one authorised actor can include or materialise newer Lerna evidence in a Hydra snapshot before the contestation deadline.
