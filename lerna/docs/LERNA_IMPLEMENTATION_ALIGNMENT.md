# Lerna Thesis Boundary

This note records the intended thesis positioning.

## Decision

The paper presents Lerna as one factory-rights protocol with two settlement
safety envelopes:

- Head-local factory settlement.
- Claim-materialisation settlement.

They are two ways of placing the same compact factory-rights state inside a
host ledger environment.
The Head-local profile is lighter, but it is safe only when the latest Lerna
state is available in signed Head evidence before close or can be used for
contest. The claim-materialisation profile is heavier, but it makes repayment of
compact rights explicit at the ledger boundary.

Instantiation evidence may be used as corroborating evidence for the
claim-materialisation profile. The thesis claim should remain generic: ledger
objects, transition invariants, acceptance obligations, and deployment
assumptions.

## Paper Surface

The paper should emphasise:

- Lerna below Interhead: virtual rights inside physical or virtual Heads;
- Lerna and Hydrozoa: deterministic inner materialisation before outer rollout;
- why compact factory state matters for Hydra fanout pressure and finalisation
  hazards;
- Head-local factory state, snapshot embedding, and conditional release safety;
- compatibility with physical Heads, Interhead virtual Heads, and
  Hydrozoa-style ledgers;
- what Lerna deliberately does not take from Hydra, Interhead, Hydrozoa, eltoo,
  factories, and Perun;
- inner factory rights discipline;
- deterministic inner materialisation before outer fanout or rollout;
- factory-local non-interference plus release receipts;
- host-interface obligations, no-stranding, and balance preservation;
- claim state and transition fields at a protocol level;
- latest-state domain, network, Head, factory, and epoch binding;
- exact full authorisation or reduced-signature non-interference;
- exact lovelace and native-asset conservation;
- release-receipt nullifiers;
- partial-batch continuation and final zero-claim acceptance;
- materialisation admission bounds;
- proof availability;
- evidence binding between ledger observations and accepted transitions.

## Boundary

The paper remains conditional on Hydra contestability, Cardano ledger
validation, proof availability, watcher/operator liveness, and bounded
materialisation capacity. The Head-local profile additionally depends on signed
snapshot availability for the latest Lerna state. Instantiation evidence should
support these claims without becoming the narrative centre of the thesis.
