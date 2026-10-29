# PRD — fontspace

Date: 2026-10-29 · Level 3 · Target: 2026-12-18

Requirement IDs are stable. `[MVP]` ships by the target date; `[STRETCH]` ships only if time
allows; `[CUT-N]` marks position in the cut order.

---

## 1. Font 1 — Cursive (connecting, lowercase)

### Character set

| ID | Requirement | Priority |
|---|---|---|
| F1-01 | Lowercase `a`–`z`, each drawn from the owner's handwriting | MVP |
| F1-02 | `.hi` entry-variant for every lowercase letter that needs one after a high exit | MVP |
| F1-03 | `space` | MVP |
| F1-04 | Non-connecting punctuation: `.` `,` `'` `-` `!` `?` | MVP |
| F1-05 | Capitals `A`–`Z`, non-connecting, shared with Font 2 | MVP |
| F1-06 | Digits `0`–`9`, non-connecting, shared with Font 2 | MVP |
| F1-07 | French accents `é è ê à ç`, built as components over base letters | MVP |
| F1-08 | Extended accents `ë î ï ô ù û â œ` and guillemets `« »` | STRETCH / CUT-1 |
| F1-09 | `.notdef` that is visibly a missing-glyph box, not blank | MVP |

### Joining behaviour

| ID | Requirement | Priority |
|---|---|---|
| F1-10 | Exactly **two** join heights: `join_low` (~40–50% x-height) and `join_high` (~x-height) | MVP |
| F1-11 | Every lowercase glyph has an entry stub beginning exactly at `(0, entry_height)` | MVP |
| F1-12 | Every lowercase glyph has an exit stub terminating exactly at `(advance_width, exit_height)` | MVP |
| F1-13 | `b`, `o`, `v`, `w` exit at `join_high`; all other lowercase exit at `join_low` | MVP |
| F1-14 | **All 676 ordered bigrams connect with no feature code applied** — the geometric contract | MVP |
| F1-15 | `calt` substitutes the `.hi` entry form after any member of `@HIGH_EXIT = [b o v w]` | MVP |
| F1-16 | Uniform, fixed sidebearings across all connecting lowercase glyphs | MVP |
| F1-17 | **No kerning pairs between connecting glyphs** | MVP |
| F1-18 | Kerning permitted only among non-connecting glyphs (capitals, digits, punctuation) | MVP |
| F1-19 | Word-initial / word-final positional forms (`.init` / `.fina`) | STRETCH |
| F1-20 | Drawn per-pair exception forms, where flagged during proof review | STRETCH / CUT-3 |
| F1-21 | Ligatures, only where two letters genuinely share one drawn stroke | STRETCH / CUT-3 |

### Explicitly rejected

| ID | Rejected | Reason |
|---|---|---|
| F1-X1 | GPOS `curs` cursive attachment | Unreliable for Latin outside HarfBuzz; repositions but cannot reshape strokes |
| F1-X2 | `rclt` instead of `calt` | Patchier real-world app support; no benefit here |
| F1-X3 | Separate entry/exit/connector stroke glyphs | Too much machinery for the timeline |
| F1-X4 | Single join height for all letters | Produces the documented `b o v w` dipping failure |

---

## 2. Font 2 — Simple (non-connecting)

| ID | Requirement | Priority |
|---|---|---|
| F2-01 | Uppercase `A`–`Z`, handdrawn | MVP |
| F2-02 | Lowercase `a`–`z`, handdrawn, non-connecting | MVP |
| F2-03 | Digits `0`–`9` | MVP / CUT-4 |
| F2-04 | Basic punctuation, matching Font 1's set | MVP / CUT-4 |
| F2-05 | Capitals and digits drawn to harmonise with Font 1 in weight and slant, so Font 1 can borrow them | MVP |
| F2-06 | Conventional spacing and kerning (unconstrained by joins) | MVP |
| F2-07 | French accents matching Font 1's set | STRETCH / CUT-2 |

Font 2 is the designated schedule buffer — CUT-5 is Font 2 in its entirety.

---

## 3. Combination tracker

| ID | Requirement | Priority |
|---|---|---|
| T-01 | Single source of truth, version-controlled, human-readable, machine-parseable | MVP |
| T-02 | One row per ordered bigram: **676 rows** | MVP |
| T-03 | Boundary rows: 26 word-initial + 26 word-final = **52 rows** | MVP |
| T-04 | Each row carries a status from a defined lifecycle, defaulting to "default is fine" | MVP |
| T-05 | Each row records its exception mechanism (`none` / `calt` / `liga`) and exception glyph name | MVP |
| T-06 | Each row links to the paper sheet ID the letters came from | MVP |
| T-07 | Each row records review date and free-text notes | MVP |
| T-08 | **The tracker records potential exceptions. It is not a to-do list.** Coverage reporting counts cells *reviewed*, never cells *completed* | MVP |
| T-09 | Trigram rows, added only when a specific case is flagged | STRETCH |

### Tools the tracker feeds (built in later phases, not Phase 0)

| ID | Requirement | Priority |
|---|---|---|
| T-10 | `.fea` generator — emits `calt` and ligature code from rows where mechanism ≠ `none` | MVP |
| T-11 | Proof-sheet renderer — renders all 676 combinations for visual review | MVP |
| T-12 | Coverage report — counts rows by status, flags unreviewed cells | MVP |
| T-13 | Join validator — asserts entry/exit node coordinates in every UFO glyph | MVP |

---

## 4. Build and output

| ID | Requirement | Priority |
|---|---|---|
| B-01 | Sources are UFO + designspace, text-based, committed to git | MVP |
| B-02 | OpenType features live in `.fea` text files, generated where possible | MVP |
| B-03 | One-command build producing OTF, TTF and WOFF2 for both fonts | MVP |
| B-04 | Build is reproducible on Windows with a pinned Python environment | MVP |
| B-05 | Font metadata: family name, style, version, designer, licence | MVP |
| B-06 | Autohinting via build defaults; no manual hinting | MVP |

---

## 5. Testing — the definition of "works"

| ID | Tier | Requirement |
|---|---|---|
| V-01 | **A** | All 676 bigrams connect visually in Chrome |
| V-02 | **A** | All 676 bigrams connect visually in Word with contextual alternates **off** |
| V-03 | **A** | Both fonts install cleanly on Windows |
| V-04 | **A** | Every glyph passes the join-coordinate validator |
| V-05 | B | Every shipped exception fires in Chrome, Figma, Adobe, and Word with `calt` on |
| V-06 | B | `hb-shape` assertions confirm expected glyph substitutions, not just visual review |
| V-07 | C | Accented glyphs render correctly in all target environments |
| V-08 | C | WOFF2 loads and renders in Chrome, Firefox and Safari |

Google Docs is out of scope — it accepts no custom fonts.

---

## 6. Paper process

| ID | Requirement | Priority |
|---|---|---|
| P-01 | Printable template sheet with baseline, x-height, cap-height, ascender, descender guides | MVP |
| P-02 | Template shows both join heights as dashed lines, plus edge tick marks per glyph box | MVP |
| P-03 | Template includes a slant guide at the measured angle of the owner's hand | MVP |
| P-04 | Every glyph box is labelled with sheet ID and glyph name (e.g. `S01-a`) | MVP |
| P-05 | Documented scan settings; scans archived in-repo as process evidence | MVP |
| P-06 | Documented cleanup standard, including a node-count ceiling | MVP |
| P-07 | Documented glyph and file naming conventions | MVP |
| P-08 | Dated journal of decisions and drawing rounds | MVP |

---

## 7. Dependencies on the owner

| ID | Needed | Blocks |
|---|---|---|
| O-01 | Handwriting samples | Template sheet (P-01…P-03), all drawing work |
| O-02 | Toolchain sign-off ($0 stack; $49 fallback) | Phase 1 spike `S1` |
| O-03 | Aesthetic direction — slant, weight, loop style, formality | Drawing round 1 |
| O-04 | Approval of the final 676-bigram proof | Ship |
