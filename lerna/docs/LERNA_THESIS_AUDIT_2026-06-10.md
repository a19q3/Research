# Lerna Thesis Audit

Date: 2026-06-10

Scope: academic rigour and model soundness of the Lerna thesis package after
recentring it as a generic protocol argument.

## Executive Verdict

The thesis is strongest when presented as a protocol paper about compact
factory-rights settlement, not as a status report. Lerna should
be read as one protocol with two settlement safety envelopes:

- Head-local settlement, safe only under a signed-snapshot availability
  assumption.
- Claim-materialisation settlement, safe under ledger validation, evidence
  binding, proof availability, watcher/operator liveness, and bounded
  materialisation capacity.

This framing is soundest when the paper keeps instantiation evidence in a
supporting role: useful for confidence, but not the main claim.

## What Is Sound

- The profile split prevents the overclaim that compact Head-local accounting is
  automatically safe after close.
- The claim-materialisation profile has a clear safety shape: scope binding,
  strict latest-state supersession, exact conservation, authorisation,
  non-interference, release receipts, continuation, no-stranding, and bounded
  admission.
- The comparison with Hydra, Interhead, and Hydrozoa is directionally coherent:
  Hydra supplies the outer Head lifecycle, Interhead supplies possible virtual
  Head contexts, and Hydrozoa motivates deterministic rollout before custody is
  exposed.
- The limitations are explicit enough to keep the thesis from becoming a broad
  protocol-proof claim.

## What Still Needs Tightening

- Head-local settlement needs a more formal transition relation if it is to
  carry theorem-level weight.
- Proof availability is currently an assumption: the paper should specify who
  stores proof bodies, how they are retrieved, and what failure means.
- Non-interference should remain factory-local unless the paper gives a full
  semantic model for arbitrary application state.
- Watcher/operator liveness is assumed, not guaranteed by the protocol.
- Dynamic membership, cross-Head composition, mint/burn policy logic, and
  arbitrary application semantics remain outside the accepted profile.

## Required Improvements Before Strong Academic Submission

1. Keep project-status language out of the thesis unless it refers to a
   protocol-level release receipt or released right.
2. Add a compact table mapping each protocol obligation to the corresponding
   invariant: scope, supersession, conservation, authorisation,
   non-interference, receipts, continuation, admission, proof availability, and
   final acceptance.
3. Formalise the Head-local profile or describe it as an architectural profile
   with conditional safety only.
4. Add a precise proof-availability subsection.
5. Expand related work with stable citations and a comparison matrix.
6. Keep instantiation evidence generic: describe evidence classes and boundaries
   rather than operational mechanics.

## Bottom Line

The thesis should be judged as a bounded protocol argument. Its strongest form
is that compact factory rights are safe only when they can be deterministically
materialised under explicit settlement assumptions, without stranded value or
ambiguous fallback. That is the academic line to preserve.
