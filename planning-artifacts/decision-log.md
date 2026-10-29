# Decision Log — fontspace

Every Phase 0 decision, its reasoning, what was rejected, and whether it needs the owner's
sign-off. `⚠️ NEEDS APPROVAL` marks items waiting on Jaxon.

Status key: **Decided** = made on the owner's behalf, reversible. **⚠️ Needs approval** = the
owner's call. **Open** = unresolved, deliberately deferred.

---

## D-01 · Project framing — personal creative project
**Decided.** Judged against the owner's own standards. Finished fonts outrank documentation;
docs exist to keep the work coherent.
**Rejected:** academic/honours framing with process as a co-equal deliverable — explicitly
dropped by the owner.

## D-02 · Target date 2026-12-18 (~7 weeks)
**Decided**, from the owner. Drives every scope decision below.
**Risk:** two fonts, one a connecting cursive, by a beginner, in 7 weeks is aggressive. Mitigated
by a published cut order (D-16), not by optimism.

## D-03 · Tool budget — spend $0 of the $100 ceiling
**⚠️ NEEDS APPROVAL.** Recommendation: Fontra Pak + fontmake + HarfBuzz, all free.
**Reasoning:** the font is largely *generated* — hundreds of `calt` rules emitted from the
tracker — so UFO and `.fea` must be text, in git, writable by scripts. Fontra is the only
in-budget editor whose **native** format is UFO.
**Rejected:** FontCreator 16 Essentials ($49, friendliest UI and best connecting-script tutorial,
but proprietary binary project file and UFO as interchange only; interactive proofing is
Professional-only at $199, over budget) · FontLab 8 (~$459, over budget) · FontForge (free,
brutal curve) · Glyphr Studio (free, no UFO, weak feature control).
**Kept as escape hatch:** FontCreator Essentials at $49, if spike S1 fails. ⚠️ approve this too.
**Caveat:** FontCreator pricing conflicts across sources ($49/$199 chart vs $149 third-party).
Verify on the vendor store before buying anything.

## D-04 · Source of truth is UFO3 + designspace
**Decided.** One `.glif` per glyph, text, clean diffs, agent-writable.
**Rejected:** any editor-native binary project file (kills review and scripting) · SVG-only
workflow (no font metadata) · building straight to TTF with no source layer.

## D-05 · Tracker format is CSV
**Decided.** 676 near-identical rows; CSV gives one-line-per-cell diffs, so a review session
shows in git as exactly the rows that changed.
**Rejected:** YAML (~5× the lines) · JSON (reformats noisily) · SQLite (binary blob, defeats
review) · a spreadsheet (not diffable, not scriptable).

## D-06 · Joining model — geometric defaults, not feature-driven defaults
**Decided.** Every lowercase glyph carries entry and exit stubs terminating at exact coordinates.
All 676 bigrams therefore connect **with zero feature code**.
**Reasoning:** this satisfies the brief's "rules, not per-pair drawing" requirement through the
drawing constraint itself — cheaper and far more robust than satisfying it with lookups. Decisive
side benefit: the font still connects in Word with contextual alternates off (D-12).
**Rejected:** the `.start`/`.between`/`.end` positional-alternate model (the FontCreator tutorial
method) as the *primary* mechanism — it makes correctness depend on lookups firing, which fails in
Word by default.

## D-07 · Two join heights
**Decided.** `join_low` ≈ 40–50% x-height (default) and `join_high` ≈ x-height (exits of
`b o v w`).
**Reasoning:** a single height is the documented #1 cursive-font tell — `b o v w` naturally exit
high, and forcing them down makes them dip unnaturally toward the baseline. Two heights fix that
for one class-wide `calt` rule.
**Rejected:** one height (the dipping artefact) · three or more (multiplies alternate sets and
review burden past a 7-week budget) · separate entry/core/exit stroke glyphs (most flexible, far
too much machinery).
**Revisit if:** round-1 proofs show a third height is unavoidable.

## D-08 · `calt` as the exception feature; `curs` and `rclt` rejected
**Decided.** `calt` is on by default in browsers, Adobe and Figma.
**Rejected:** GPOS `curs` cursive attachment — HarfBuzz implements it properly, but support is
unreliable in DirectWrite/Word and Adobe for *Latin*, and it only *translates* glyphs: it cannot
reshape a badly-angled exit, so it does strictly less than geometry + `calt` · `rclt` — nominally
always-on, patchier real-world support, no benefit here.

## D-09 · Contextual alternates are the default exception mechanism; ligatures are rare
**Decided.** `calt` when "this letter should look different here." Ligature only when "these two
letters are one drawing."
**Reasoning:** ligatures break text selection and cursor granularity inside the pair, kill kerning
across it, and can't be selectively reverted by the user.

## D-10 · No word-positional forms in the MVP
**Decided.** Because every letter already carries entry and exit stubs, words begin with a natural
lead-in and end with a natural exit — these read as correct cursive. `.init` / `.fina` become a
stretch refinement.
**Reasoning:** largest single scope saving available, at near-zero visual cost.
**Note:** when added later, make the initial form the *default* glyph and substitute to the
connecting form when preceded by a letter — `.fea` has no negative lookbehind, so that's the
expressible direction.

## D-11 · Context scope — bigrams plus word boundaries
**Decided.** 676 bigrams + 52 boundary positions = 728 review cells, all in scope. Trigrams
(17,576) only by individual flag during proof review.
**Rejected:** exhaustive trigram coverage.

## D-12 · Google Docs out of scope; Word supported with `calt` off
**Decided.** Google Docs accepts no custom fonts outside enterprise Workspace administration —
a platform limitation, not an engineering gap. Word has `enableOpenTypeFeatures` off by default,
so contextual alternates require ticking Font ▸ Advanced ▸ *Use Contextual Alternates* — but
geometric defaults mean **Word still connects every letter without it.** Only refinements are lost.
**Definition of "works":** Tier A = all 676 connect in Chrome *and* in Word with `calt` off.

## D-13 · Capitals and digits are borrowed from Font 2, non-connecting
**Decided.** Omitting them renders `.notdef` boxes, which looks broken. One drawing effort serves
both fonts.
**Consequence:** Font 2's capitals and digits must harmonise with the cursive in weight and slant
— a stated constraint on Font 2, not an afterthought.
**Rejected:** connecting capitals (large extra scope, and real cursive capitals often don't
connect anyway) · omitting them entirely.

## D-14 · French accents in scope — `é è ê à ç` for MVP
**Decided.** Accents sit above the letter and never touch the join, so they carry near-zero risk
to the connection system and build as components over existing bases.
**Stretch:** `ë î ï ô ù û â œ` plus guillemets `« »`.
**Rejected:** full Latin Extended.

## D-15 · No kerning in the cursive font; fixed uniform sidebearings
**Decided.** Kerning translates glyphs horizontally and tears joins. The exit stub *is* the letter
spacing, so spacing is tuned by redrawing stub length, never by nudging sidebearings. Kerning is
permitted only among non-connecting glyphs.

## D-16 · Published cut order
**Decided.** If the date is threatened, cut in this order:
1. Font 1 stretch accents → 2. Font 2 accents → 3. all drawn exceptions (defaults still work) →
4. Font 2 digits/punctuation → 5. Font 2 entirely.
**Never cut:** the 26 connecting lowercase letters, the two join heights, the `.hi` alternate set,
the Tier A proof.

## D-17 · Manual tracing, not auto-trace
**Decided.** Trace by hand in Fontra over the scan as a glyph background image.
**Rejected:** Inkscape Trace Bitmap as the primary path — on inked pen texture it yields outlines
with hundreds of ragged nodes, fatal when joins must land on an exact coordinate and stub shapes
must match across 26 glyphs. Retained as a read-only underlay for checking proportions.

## D-18 · Scan at 600 dpi grayscale, automatic adjustments off
**Decided.** Bitonal destroys the soft stroke edges needed for faithful tracing; auto-contrast and
auto-crop break comparability and the known physical scale.

## D-19 · The join validator is a first-class deliverable
**Decided.** `validate_joins.py` asserts every connecting glyph's entry/exit node coordinates,
uniform sidebearings, and a node-count ceiling, against the UFO, pre-build.
**Reasoning:** off-coordinate hand-drawn joins are risk #1 — the one high-severity risk that is
cheap to eliminate automatically and expensive to catch by eye.

## D-20 · The tracker is not a to-do list
**Decided**, per the owner's explicit framing. `ok-default` is a terminal success state and the
expected destination for the large majority of the 676. The coverage report counts cells
**reviewed**, never cells **completed**. A tracker that is 100% `ok-default` describes a finished
font.

## D-21 · Lean artifact set — six docs, not seven
**Decided.** The paper-process spec and template-sheet spec are folded into the architecture doc
(§8) rather than spun out; the BMAD UX-design workflow is skipped because the template sheet *is*
the UX here.
**Reasoning:** owner asked explicitly not to overengineer the process.

---

## Open items

| ID | Item | Blocks | Owner |
|---|---|---|---|
| O-01 | **Handwriting samples** — join heights, x-height and slant angle must be measured from the owner's actual hand before `joins.yaml` has real values | template sheet, all drawing | ⚠️ Jaxon |
| O-02 | **Toolchain sign-off** — approve the $0 stack, and approve FontCreator Essentials ($49) as the fallback if spike S1 fails | Phase 1 | ⚠️ Jaxon |
| O-03 | **Aesthetic direction** — loops vs. minimal, slant, stroke weight, formal vs. casual. Deferred until samples arrive | drawing round 1 | ⚠️ Jaxon |
| O-04 | Font naming — real family names for both fonts | build metadata | ⚠️ Jaxon |
| O-05 | Whether `e` needs an early dedicated alternate (documented distortion risk) | drawing round 1 review | Open — decide from proofs |
| O-06 | Git — repo is not yet initialised | version control of sources | Open, trivial |
