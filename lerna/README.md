# Lerna Protocol

This directory contains the Lerna paper package. The paper presents Lerna as
one Hydra-native factory-rights protocol with two settlement safety envelopes:
Head-local factory settlement and claim-materialisation settlement.

- `paper.tex` is the editable source.
- `paper.pdf` is the reading copy.
- `docs/LERNA_IMPLEMENTATION_ALIGNMENT.md` records the thesis boundary.

The thesis is intentionally generic. It states the protocol in terms of ledger
objects, transition invariants, evidence binding, and explicit assumptions.
Instantiation evidence may corroborate the claim-materialisation profile, but it
is not the centre of the paper.

The paper deliberately uses settlement profiles rather than a delivery story.
The Head-local profile is valid only when an authorised participant, operator,
or watcher can ensure the latest Lerna state is in a signed Hydra snapshot
before close, or can contest a stale close with an already-signed newer
snapshot. Otherwise the claim-materialisation profile is the intended safety
envelope.
