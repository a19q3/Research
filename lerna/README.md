# Lerna Protocol

This directory contains the Lerna paper package. The paper now presents Lerna
as one Hydra-native factory-rights protocol with two settlement profiles:
Head-local factory settlement and claim-materialisation settlement.

- `paper.tex` is the editable source.
- `paper.pdf` is the reading copy.
- `docs/LERNA_IMPLEMENTATION_ALIGNMENT.md` records the implementation mapping.

Current implementation evidence: the claim-materialisation profile is checked
by the Aiken validator, deterministic model, real Hydra/Cardano devnet verifier,
profile matrix, local file hashes, and executable security-theorem gate. Fresh
local reproduction requires regenerating or path-normalising recorded
Hydra/Cardano artifacts for the current checkout.

The paper deliberately uses settlement profiles rather than a roadmap story.
The Head-local profile is valid only when an authorised participant, operator,
or watcher can ensure the latest Lerna state is in a signed Hydra snapshot
before close, or can contest a stale close with an already-signed newer
snapshot. Otherwise the claim-materialisation profile is the intended safety
envelope.
