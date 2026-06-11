# Lerna AI-Prose Audit

Date: 2026-06-11

Scope: `lerna/paper.tex`, with emphasis on passages that could read as
machine-generated, over-polished, or rhetorically padded. This audit concerns
style and academic presentation only; it does not replace the model-soundness
audit in `LERNA_THESIS_AUDIT_2026-06-10.md`.

## Verdict

The paper was not generically incoherent, but it had a recognisable
AI-prose surface in its front matter and positioning sections: repeated
boundary phrases, slogan-like abstractions, and self-descriptive prose about
what the thesis is or is not. The definitions, assumptions, lemmas, theorem,
and review matrix were materially stronger and needed lighter treatment.

The first edit pass removes the strongest signals without weakening the
technical boundary. The paper now reads more like a protocol paper and less
like a generated positioning memo.

## Method

- Read the full paper.
- Searched for repeated meta-discourse and stock qualifiers:
  `intentionally`, `deliberately`, `useful`, `modest`, `stronger`,
  `appropriate`, `design ethic`, `central claim`, `not the thesis`, and
  repeated `Lerna takes...` constructions.
- Searched for repeated thesis slogans and defensive boundary terms:
  `safety envelope`, `compactness debt`, `inner rule`, `outer settlement`,
  `not a replacement`, `not a proof`, `corroboration`, and `rather than`.
- Edited only high-confidence prose issues where the technical claim was
unchanged.

## Fixed High-Confidence Issues

1. Front-matter over-positioning.

   The abstract previously used a dense sequence of generic framing moves:
   "not a replacement", "missing inner rule", "intentionally bounded",
   "does not compete", "originality is narrower", and "appropriate safety
   envelope". This made the opening sound defensive and synthetic.

   Current status: rewritten at `paper.tex` lines 123-148 to state the protocol
   object, profiles, assumptions, and host relationship directly.

2. Slogan overuse around compactness.

   The old text repeated "compactness is debt", "makes compactness payable",
   "debt is paid", and "ledger debt". The metaphor was memorable but too
   insistent for academic prose.

   Current status: replaced with "settlement obligation" language at
   `paper.tex` lines 247-300. The model claim is unchanged: compact rights are
   acceptable only when there is a deterministic materialisation path.

3. Delivery-story and competitor disclaimers.

   The introduction previously spent too much effort saying that Lerna was not
   a Hydra replacement, Hydrozoa competitor, or eltoo construction. Some
   boundary-setting is necessary, but repeated disclaimers create an AI-like
   "not X but Y" rhythm.

   Current status: reduced in the introduction at `paper.tex` lines 189-194.
   A concise related-work boundary remains at line 1234, where it is
   appropriate.

4. Repetitive prior-work formulae.

   The prior-work boundary subsection had four adjacent items beginning with
   "Lerna takes...". That repetition sounded generated.

   Current status: rewritten under `Boundaries with Prior Work` at `paper.tex`
   lines 539-562.

5. Inflated positioning terms.

   Phrases such as "design ethic", "deliberately modest", "easiest to audit",
   "intentionally redundant", and "should be read as" added polish without
   adding precision.

   Current status: replaced with concrete protocol language at `paper.tex`
   lines 449-461, 533-537, 840-843, 942-947, 1021-1025, and 1131-1135.

6. Evidence-section meta-language.

   The evidence section previously said that implementation details were useful
   but "not the thesis". That was accurate but sounded like generated
   self-commentary.

   Current status: rewritten at `paper.tex` lines 1163-1191.

7. Conclusion recap inflation.

   The conclusion repeated the abstract-level positioning against Interhead,
   Hydrozoa, and Hydra, then used a broad "remaining work follows directly"
   formulation.

   Current status: shortened at `paper.tex` lines 1327-1338 to restate the two
   profiles, the narrow academic claim, and the open work.

## Terms Retained Deliberately

- `safety envelope`: retained where it denotes the profile boundary; it should
  not be expanded further.
- `inner` / `outer`: retained where it distinguishes factory materialisation
  from Hydra fanout or Hydrozoa-style rollout.
- `no-stranding`: retained as an invariant label.
- `proof availability`: retained as a protocol assumption.
- `bounded continuation`: retained as a transition property.

These terms are repeated because they carry the paper's model, not because the
prose needs rhetorical padding.

## Remaining Human-Review Targets

- The abstract is now cleaner, but it is still concept-dense. A final
  submission pass should decide whether to split the second abstract paragraph
  or remove one related-work sentence.
- The phrase `safety envelope` remains visible. It is defensible as a defined
  profile concept, but an external reviewer may prefer `profile` or
  `acceptance boundary` in some non-definition passages.
- The paper still relies on long lists of invariants. That is technically
  appropriate, but tables should carry more of the list burden if the prose
  feels repetitive after typesetting.
- The Head-local profile still sounds more architectural than formal. That is a
  model issue rather than an AI-prose issue; it should be handled through
  formalisation or explicit demotion, not stylistic smoothing.

## Post-Edit Signal Check

The source paper no longer contains the searched stock qualifiers outside
technical terms. The main stock phrases removed or reduced were:
`intentionally`, `deliberately`, `design ethic`, `compactness debt`,
`compactness useful`, `Lerna takes...`, `not the thesis`, `should be read as`,
and `easiest to audit`.

The remaining prose is not free of house style, but it is now much less likely
to be read as AI-generated padding.
