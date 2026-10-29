# Product Brief — fontspace

Date: 2026-10-29 · Owner: Jaxon · Target: 2026-12-18 · Phase: 0 (planning)

## What this is

Two original typefaces, both drawn by hand on paper and finished as real, installable fonts
(OTF/TTF plus WOFF2 for web).

- **Font 1 — "Cursive"**: a connecting cursive, lowercase, derived from the owner's own
  handwriting. Every letter joins its neighbours. Every one of the 676 lowercase pairs must
  render with a working connection by default.
- **Font 2 — "Simple"**: a non-connecting handdrawn face, basic uppercase and lowercase.

A personal creative project, judged against the owner's own standards. The finished fonts are
the point; documentation exists to keep the work coherent, not as a deliverable in its own right.

## Why it's interesting as an engineering problem

A connecting cursive is a combinatorics problem wearing a typeface costume. 26 letters produce
676 ordered pairs, and a font that only looks right on the pairs its designer happened to test
is the normal failure mode — the thing that makes most script fonts look wrong the moment you
type an unusual word.

The approach here inverts the usual one: rather than drawing good-looking pairs, define a
geometric contract every letter must satisfy, so that **correctness is structural and
exceptions are optional polish**. Exceptions then get tracked systematically across all 676
cells rather than discovered by accident.

## Goals

1. Two fonts that install and render correctly on Windows, in browsers, and in Word.
2. Font 1 connects **every** lowercase pair by default — no gaps, no broken joins, no pair that
   only works by luck.
3. Handwriting that still reads as the owner's, not as a generic script face.
4. A combination tracker that makes exception coverage legible at a glance.
5. A repeatable paper → scan → trace → build pipeline, so a second drawing round is cheap.

## Success criteria

| # | Criterion | How it's verified |
|---|---|---|
| S1 | All 676 bigrams connect visually | Proof sheet of all 676, reviewed; Tier A test |
| S2 | Connections survive in Word with contextual alternates **off** | Manual check in Word |
| S3 | Both fonts install on Windows and render in Chrome, Figma, Word | Manual check per environment |
| S4 | Every glyph passes the join-coordinate validator | Automated check against UFO sources |
| S5 | Owner recognises their own handwriting in a paragraph of set text | Owner's judgement — the real bar |
| S6 | A revised letter can be redrawn, rescanned and rebuilt in under an hour | Timed once, in Phase 3 |

## Non-goals

- Commercial release, licensing, distribution, or a specimen/marketing site.
- Variable fonts, multiple weights, or italics.
- Capitals *in connecting form* in Font 1 — capitals are borrowed non-connecting from Font 2.
- Full Latin Extended coverage. French accents are the only non-ASCII in scope.
- Google Docs support. It accepts no custom fonts; out of scope by platform limitation.
- Hinting beyond autohint defaults.
- Exhaustive trigram or longer-context coverage.
- Kerning the cursive font. Kerning tears joins; it is deliberately excluded.

## Constraints

- **Platform:** Windows.
- **Budget:** ≤ $100 for tooling. Current plan spends **$0**.
- **Skill:** owner is new to type design. Tooling choices favour a shallow learning curve and
  automation over manual repetition.
- **Division of labour:** the owner draws on paper and makes all aesthetic calls. Agents handle
  tooling, feature code, proofs, validation, and tracking.
- **Timeline:** ~7 weeks. This is the dominant risk; see the cut order in the research report.

## Known scope risk, stated plainly

Two fonts — one of them a connecting cursive — in seven weeks, by someone new to type design, is
aggressive. It is achievable *only* because the geometric join model removes the need for
hundreds of hand-drawn pair exceptions. If anything slips, Font 2 and all optional exceptions are
the designated buffer; the 26 connecting lowercase letters are not.

## Open items needing the owner

1. **Handwriting samples** — needed before the template sheet can be finalised (join heights and
   slant angle are measured from the owner's actual hand).
2. **Toolchain sign-off** — the recommendation is $0; confirm the free stack is acceptable and
   that FontCreator Essentials ($49) is approved as a fallback if the Phase 1 spike fails.
3. **Aesthetic direction** — loops vs. minimal, slant angle, stroke weight, formal vs. casual.
   Deferred until samples are in hand.
