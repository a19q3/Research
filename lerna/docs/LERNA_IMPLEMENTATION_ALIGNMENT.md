# Lerna Implementation Alignment

This note records why the paper was updated on 6 June 2026.

## Decision

The paper now follows the implementation in `a19q3/Lerna` at commit
`66ff116`. The production target is the claim-UTxO settlement discipline:

- Aiken claim validator.
- Deterministic off-chain model.
- Real Hydra/Cardano devnet evidence.
- Reduced-signature, native-asset, and multi-batch profile matrix.
- Executable security-theorem gate.

Historical Head-local acceptance notes were removed from this paper package
because they describe a different target and make the current claim ambiguous.

## Paper Surface

The paper should describe:

- claim datum and redeemer fields;
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
