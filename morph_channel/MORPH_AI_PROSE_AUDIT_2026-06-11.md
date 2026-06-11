# Morph Channel AI-Prose Audit

Date: 2026-06-11

Scope: `morph_channel/paper.tex`, reviewed at the current workspace state.
This audit focuses on passages that could read as machine-generated,
over-polished, defensive, or rhetorically padded. It does not replace a
protocol-soundness audit.

## Verdict

The paper does not read as generic AI output overall. Its strongest sections
are the pseudo-code, invariants, propositions, factory acceptance matrix, and
audit matrix. Those parts use concrete protocol objects and are much less
exposed to an "AI prose" reading.

The recognisable AI-like surface is concentrated in the front matter,
introduction, contribution/principle lists, factory narrative transitions,
challenge-window discussion, related work, and conclusion. The repeated pattern
is not grammatical error. It is a combination of list-heavy abstraction,
defensive "not X" boundary-setting, repeated meta-discourse about what the
paper is doing, and polished but under-specific phrases such as "familiar
tension", "changes the substrate", "not merely aesthetic", "complexity
boundary", and "necessary first step".

High-level judgement: keep the technical model, but rewrite the opening and
closing narrative so that it sounds like an author explaining a precise design,
not a generated position paper explaining its own framing.

## Method

- Read the full LaTeX source.
- Mapped the manuscript structure with line numbers.
- Searched for repeated stock framing terms:
  `This paper`, `This draft`, `This section`, `The construction`,
  `deliberately`, `intentionally`, `not merely`, `not by itself`,
  `not enough`, `not equivalent`, and `not automatically`.
- Checked repeated technical-modifier pressure:
  `conservative`, `canonical`, `bounded`, `explicit`, `ordinary`, `should`,
  `must`, `authority`, `evidence`, `value`, and `proof`.
- Treated technical repetition as acceptable where it names a protocol object,
  and suspicious where it replaces a concrete mechanism or empirical claim.

## Signal Summary

- `This paper`: 3 occurrences.
- `This draft`: 1 occurrence.
- `This section`: 2 occurrences.
- `deliberately`: 5 occurrences.
- `intentionally`: 3 occurrences.
- `not merely`: 4 occurrences.
- `not by itself`: 3 occurrences.
- `conservative`: 13 occurrences.
- `canonical`: 21 occurrences.
- `bounded`: 17 occurrences.
- `should`: 26 occurrences.
- `must`: 45 occurrences.
- `authority`: 27 occurrences.
- `evidence`: 61 occurrences.
- `value`: 72 occurrences.
- `proof`: 64 occurrences.

Most occurrences of `authority`, `evidence`, `value`, and `proof` are justified
by the protocol. The higher-risk signals are the repeated defensive qualifiers
and meta-text around paper scope.

## Highest Priority Issues

### 1. Abstract is too enumerative and self-descriptive

Lines: 91-104.

Issue: the abstract has four labelled mini-sections and long contribution
sentences. Lines 96 and 100 are especially list-heavy. The form is precise, but
it reads more like a generated executive summary than an academic abstract.

Specific signals:

- "This paper proposes..."
- "It decomposes..."
- "It defines..."
- "It gives..."
- "It also introduces..."
- "For bilateral channels..."
- "For factories..."
- "This is a research-level protocol-design paper..."

Recommended action: rewrite as three compact paragraphs:

1. State the CKB-specific problem.
2. State the construction in terms of funding bundle, State Cell, sponsor
   partition, and descriptor/proof objects.
3. State the formal consequences and limitations without the "Paper Type"
   label.

Do not remove the technical boundaries. Remove the self-labelling.

### 2. Contribution count is wrong

Lines: 196-208.

Issue: line 196 says "This paper makes eight contributions", but the list
contains C1 through C9.

Why it matters: this is a plain accuracy defect, and it also produces the
impression of generated list management.

Recommended action: either change "eight" to "nine", or merge C8 and C9 if the
author wants exactly eight. I would prefer changing it to "nine" only if all
nine items remain genuinely distinct. Otherwise, merge:

- C8: challenge-window budgeting.
- C9: factory non-interference.

Those are conceptually separate, so "nine" is probably the smaller correction.

### 3. Introduction uses stock positioning phrases

Lines: 118-132.

Issue: the introduction opens with broad academic framing rather than the
paper's concrete design pressure. The phrases are defensible individually, but
together they sound polished and generic.

Examples:

- line 118: "motivated by a familiar tension"
- line 120: "CKB changes the substrate"
- line 126: "not a direct port"
- line 132: "intended audit scope is construction audit"

Recommended action: start from the concrete mismatch:

- Bitcoin-style channel updates reshape spend paths over a funding output.
- CKB Cells can carry typed state and stable outpoints.
- Morph separates the value object, the dispute evidence, and the fee payer.

Move the "construction audit" language to the appendix or remove it from the
main introduction.

### 4. Problem statement sounds like a taxonomy generated after the fact

Lines: 167-190.

Issue: the three subsection titles are useful, but the prose relies on abstract
problem labels: "coupling problem", "fee-boundary problem", "authority object
problem", "truth objects", and "collapses authority". This creates a tidy
taxonomy, but the missing ingredient is a concrete failure mode per problem.

Recommended action: for each problem, give one CKB transaction-level failure
example before naming the abstraction. For example:

- stale state evidence controls a later vault set;
- sponsor change accidentally absorbs channel capacity;
- a descriptor-shaped Cell is mistaken for state authority.

Then keep the problem labels as summaries, not as the primary explanation.

### 5. Design principles are too numerous and slogan-like

Lines: 210-227.

Issue: fourteen principles in a row create an "LLM requirements list" surface,
even though many items are technically correct. Several are really invariants
or acceptance requirements rather than design principles.

Recommended action: split them into:

- 4-5 actual design principles near the start;
- formal invariants in the relevant technical sections;
- deployment/test requirements in the evaluation agenda or audit matrix.

Good candidates to keep as principles:

- script-level deployability;
- no hidden fee leakage;
- narrow descriptors;
- envelope before factory body;
- splice freshness.

Move the rest to invariants or acceptance tests.

## Section-by-Section Audit

### Front Matter and Abstract

Lines 91-104: high rewrite priority.

The labelled abstract is useful for internal review, but for an academic paper
it over-explains its own structure. The wording is also overloaded with
compound nouns: "state-header-centric signing domain", "sponsor-owned
publication layer", "partition-conservation algorithm", and "envelope-first
admissibility rule". Keep those terms only if they are defined immediately
afterwards, or reduce the abstract to the core mechanism.

Line 104 is technically prudent, but the phrase "not a claim of production
completeness" sounds defensive. Better: "The construction uses only existing
CKB primitives..." and leave production readiness to the limitations section.

### Introduction and Scope

Lines 118-126: medium-high rewrite priority.

The core argument is sound, but the route into it is generic. The reader should
reach "stable funding anchor plus moving State Cell" faster.

Line 132: high rewrite priority.

"The intended audit scope is construction audit" feels like an internal note.
It should become a sentence about the paper's scope, or move to an appendix.

### Design Thesis

Lines 136-141: medium rewrite priority.

The thesis is clear, but the "X answers..." rhythm is slightly formulaic:
"Funding identity answers...", "State authority answers...", "Sponsor
authorisation answers...". Consider one direct sentence:

"Morph treats the funding object, the signed state, and the fee-paying
transaction inputs as separate authorities."

Then explain the consequence once.

### Problem Statement

Lines 171-190: medium-high rewrite priority.

"The coupling problem is not merely aesthetic" is a classic synthetic
transition. Replace it with a specific example of the bad transaction shape.
"Multiple competing truth objects" is vivid, but imprecise; use "authority
sources" or "objects accepted by scripts".

### Contributions and Principles

Lines 196-208: high priority because of the count error.

Lines 210-227: medium-high rewrite priority because the list is long and
label-heavy.

The list contains valuable ideas, but it makes the paper sound as if every
phrase has been elevated into a named doctrine. A protocol paper can keep
defined invariants, but it should avoid too many slogan labels in one place.

### System Model and Formal Objects

Lines 229-344: mostly keep.

This section is much stronger. The definitions are concrete and the repeated
terms are largely justified. Mild edits:

- line 293: "A deployable instance should..." is a little tentative. Use "A
  deployable instance expresses..." if this is a requirement.
- line 340: "The purpose is not only..." is a repeated boundary pattern. Split
  it into two factual sentences.
- line 344: sponsor-policy material is repeated later at lines 509-515 and
  1069. Keep one canonical statement and refer back to it.

### Script-Level Validation Semantics

Lines 375-515: mostly keep.

The pseudo-code is one of the paper's least AI-like parts. It has concrete
checks and clear failure boundaries.

Line 377 can be tightened:

"The following pseudo-code states the script checks required for an
implementable construction."

Lines 445-456 are good, but "deliberately" appears twice nearby. One occurrence
can be removed without losing meaning.

### Protocol State Machine and Transactions

Lines 517-632: mostly keep.

This is direct and technical. The main weak line is 593:

"The transaction body is not the protocol authority. It is a reconstructible
carrier..."

The idea is important, but the "not X, it is Y" rhythm recurs throughout the
paper. It can be made more concrete:

"Participant signatures authorise the state header; the transaction body only
chooses live inputs, sponsor inputs, and fee rate."

### Partition Conservation

Lines 634-765: keep with light edits only.

This section is concrete and well justified. Line 740 is a little explanatory,
but it earns its length because it distinguishes reserve, business CKB, xUDT,
and unrelated Cells.

No major AI-prose problem here.

### Settlement Descriptor

Lines 767-786: keep.

This section is concise. "not a general-purpose application language" is a
boundary phrase, but it is doing real work and should stay unless the paper
creates too many similar phrases elsewhere.

### Bilateral Witness Mode

Lines 788-793: medium rewrite priority.

"This is not a claim that privacy is irrelevant; it is a complexity boundary"
sounds defensive. Replace with direct engineering rationale:

"For the bilateral profile, plaintext witnesses keep the first implementation
auditable. Privacy can be added later by replacing the payload with committed
state material once size and CKB-VM costs are measured."

### Factory Proof Mode

Lines 794-900: medium rewrite priority.

The definitions are good. The surrounding prose is more polished than
necessary.

Line 796: "neither private nor obviously scalable" is vague. Say what scales:
transaction size, witness size, proof verification cycles, or participant
data exposure.

Line 855: "The difficult object in a factory is therefore not the Merkle proof
alone..." is a strong idea, but the sentence is rhetorically heavy. Make it
more formal:

"A Merkle proof is insufficient unless the transition family also accounts for
rights depending on reserve, membership, exit path, and sponsor-budget state."

Line 900: the proof is clear, but "after the fact" has a conversational tone.
Use "under a weaker authority rule" instead.

### Factory Acceptance Requirements

Lines 903-941: mostly keep.

This is one of the more credible sections because it demands devnet evidence.
Line 905 can be tightened:

"A deployment is not accepted merely because one representative factory path
succeeds."

Line 937 is good content, but "A paper proof ... is therefore not enough on its
own" is another defensive formulation. Consider:

"The non-interference proof must be accompanied by acceptance evidence..."

### Challenge-Window Semantics

Lines 943-1034: medium-high rewrite priority.

The section is technically important, but it has several slogan-like boundary
sentences:

- line 945: "not a paper constant"
- line 1012: "Safety should be defined..."
- line 1014: "This does not assume..."
- line 1034: "not merely a wallet convenience"

Recommended action: keep the content, but write in deployment terms:

- "The challenge window is a deployment parameter derived from confirmation
  depth, watchtower polling, transaction construction time, confirmation
  target, fee-bump budget, and reorganisation margin."
- "A response is safe only after the newer State Cell confirms before the
  finalisation input becomes mature."
- "Fee bumping means rebuilding the carrier transaction with current sponsor
  inputs; it does not rely on Bitcoin-style replacement semantics."

### Signature Domains and Safety Properties

Lines 1044-1105: mostly keep.

The lemmas and propositions are clear. The technical repetition is acceptable
because these are proof obligations. No major AI-prose issue.

### Watchtower and Recovery

Lines 1061-1071: medium rewrite priority.

Line 1063 is good but aphoristic: "A watchtower is a liveness delegate, not a
custodian." This is acceptable if it is the only aphorism in the section, but
the paper already has many "not X" boundary statements. Consider replacing it
with:

"A watchtower stores state packages and publishes newer evidence; it never
receives authority over channel-owned value."

Line 1069 repeats sponsor-policy constraints. Consolidate with the sponsor
partition section.

### Evaluation Agenda

Lines 1107-1124: keep.

This is concrete. It helps counter the AI-prose risk by demanding measurements
and negative tests.

### Related Work

Lines 1126-1132: medium-high rewrite priority.

The related-work section is thin compared with the paper's technical ambition.
Phrases such as "represents the penalty-based payment-channel family" and
"motivate shared funding and local exit questions" sound like placeholder
survey prose.

Recommended action: add a small comparison table:

- funding object;
- update authority;
- dispute replacement mechanism;
- fee-payer model;
- factory/local-exit support;
- required base-layer assumptions.

That would make the section feel researched rather than generated.

### Limitations

Lines 1134-1147: mostly keep.

This section is appropriately candid. The only style risk is line 1146, which
is very long. Split it for readability.

### Conclusion

Lines 1149-1153: medium-high rewrite priority.

The conclusion repeats a compressed list of the whole paper, then ends with
"necessary first step". That closing phrase is a common generated-paper
ending.

Recommended action: close with the exact claim and the exact remaining burden:

"Morph shows how to separate funding identity, state evidence, sponsor fee
responsibility, and factory-local proof admission using current CKB script
objects. The construction still requires benchmark evidence for witness size,
CKB-VM cycles, challenge-window parameters, and factory proof costs before it
can be treated as a deployable channel protocol."

## Passages That Should Probably Stay

The following areas are dense but not AI-like in the problematic sense:

- Lines 276-296: funding bundle, vault authorisation, current State Cell
  pointer, and initial uniqueness.
- Lines 308-340: canonical State Header and signing domain.
- Lines 407-443: State Cell transition pseudo-code.
- Lines 464-486: vault-spend pseudo-code.
- Lines 699-738: partition-conservation algorithm.
- Lines 887-901: factory authorisation propositions, after light wording
  cleanup.
- Lines 1036-1059: newer-state and anti-replay propositions.
- Lines 1160-1194: construction audit matrix.

These sections sound technical because they bind claims to objects, scripts,
or tests. They should guide the rewrite style for the rest of the paper.

## Rewrite Policy

Recommended rewrite order:

1. Fix the contribution count at line 196.
2. Rewrite the abstract.
3. Rewrite the introduction and remove internal "audit scope" language.
4. Reduce the design-principle list.
5. Consolidate repeated sponsor-policy paragraphs.
6. Tighten bilateral/factory/challenge-window narrative transitions.
7. Strengthen related work with a comparison table.
8. Rewrite the conclusion.

Recommended style rules:

- Prefer one concrete CKB transaction failure over one abstract problem label.
- Use "must" only when a script, signature, or deployment profile enforces it.
- Avoid repeated "not X, but Y" sentences unless the contrast is central.
- Keep named terms only when they reappear as definitions, invariants, or
  test obligations.
- Move internal review instructions to the appendix.
- Let pseudo-code, tables, and invariants carry lists; keep prose shorter.

## Bottom Line

The paper's technical core is stronger than its rhetoric. The main AI-prose
risk is not incoherence, but excessive self-framing: the manuscript repeatedly
explains its own boundaries instead of showing the boundary through script
checks, transaction examples, and acceptance tests. A focused prose pass should
make the paper sound more authored by removing meta-discourse, fixing the
contribution-count error, and turning abstract labels into concrete CKB failure
modes.
