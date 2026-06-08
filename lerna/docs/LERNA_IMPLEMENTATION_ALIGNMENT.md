# Lerna Implementation Alignment

This note records why the paper was updated on 8 June 2026.

## Decision

The paper now presents Lerna as one factory-rights protocol with two settlement
profiles, not as a software release sequence:

- Head-local factory settlement.
- Claim-materialisation settlement.

The implementation in `a19q3/Lerna` at commit `66ff116` provides executable
evidence for the claim-materialisation profile:

- Aiken claim validator.
- Deterministic off-chain model.
- Real Hydra/Cardano devnet evidence.
- Reduced-signature, native-asset, and multi-batch profile matrix.
- Executable security-theorem gate.

The Head-local profile remains part of the theory, but it is intentionally
bounded. It is safe only when at least one authorised Head participant,
operator, or watcher can checkpoint or materialise the latest Lerna state into
a Hydra snapshot before contestation expires. If that assumption is too strong,
the paper directs the design to claim-materialisation.

## Paper Surface

The paper should describe:

- Lerna below Interhead: virtual rights inside physical or virtual Heads;
- Lerna and Hydrozoa: deterministic inner materialisation before outer rollout;
- why compact factory state matters for Hydra fanout pressure and finalisation
  hazards;
- compatibility with physical Heads, Interhead virtual Heads, and
  Hydrozoa-style ledgers;
- what Lerna deliberately does not take from Hydra, Interhead, Hydrozoa, eltoo,
  factories, and Perun;
- inner factory rights discipline;
- deterministic inner materialisation before outer fanout or rollout;
- factory-local non-interference plus release receipts;
- claim datum and redeemer fields for the implementation-backed profile;
- latest-state domain, network, Head, factory, and epoch binding;
- exact full authorisation or reduced-signature non-interference;
- exact lovelace and native-asset conservation;
- release-receipt nullifiers;
- partial-batch continuation and final zero-claim acceptance;
- materialisation admission bounds;
- proof availability;
- local ledger evidence hash binding;
- theorem obligations checked by `npm run verify:security-theorem`.

## Accepted Commands

```sh
npm test
npm run verify:security-theorem
npm run check:devnet
npm run check:profiles
```

The paper remains conditional on Hydra contestability, Cardano ledger
validation, proof availability, watcher/operator liveness, and bounded
materialisation capacity.
