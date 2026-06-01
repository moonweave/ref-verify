# ref-verify

**Zero-hallucination reference verification for Claude Code.**

LLMs confidently recall paper metadata that is wrong — transposed DOIs, truncated author lists, findings attributed to papers that don't contain them. `ref-verify` enforces live verification against CrossRef, Semantic Scholar, and PubMed so every citation you add is grounded in something actually fetched, not recalled from training data.

---

## Install

```bash
npx skills add moonweave/ref-verify -g
```

---

## What it catches

| Failure mode | Example |
|---|---|
| Wrong DOI | Paper resolves to a different article entirely |
| Wrong authors | "Carpi et al. (2011)" — but that DOI is Chapter 1 by Pelrine & Kornbluh |
| Wrong year | CrossRef says 2008, you have 2011 |
| Hallucinated content | Agent says paper "shows 380% strain" — abstract doesn't contain that number |
| Near-miss citations | Abstract mentions "500% strain" — but it's a pre-strain value, not actuation output |
| Retracted paper | Paper still has a valid DOI but was retracted |

---

## Usage

Just talk to Claude naturally. The skill triggers automatically when you're doing citation work:

```
"find 3 papers on IPMC actuators showing bending strain data"
"verify these citations before I submit: [DOI list]"
"check doi 10.1002/adma.202108361 — I think I copied it wrong"
"is that actually what the Pelrine 2000 paper says?"
"audit all 42 refs in my bibliography before submission"
"add a citation for the paper where Mirfakhrai et al. show IPMCs work underwater"
```

Does **not** trigger for general questions, prose editing, or reference formatting.

---

## Two modes

The skill picks the right mode automatically based on your task.

### Quick Screen
*For spot-checking DOIs you already have.*

Hits CrossRef, confirms title + author match, verifies DOI resolves. One line per reference, takes seconds.

```
Shahinpoor & Kim (2001) 10.1088/0964-1726/10/4/327 — PASS
Bar-Cohen (2004)        10.1117/3.547465            — WARN (editor, not author)
Carpi et al. (2011)     10.1016/B978-0-08-047488-5.00001-0 — MISMATCH (resolves to different paper)
```

### Full Audit
*For literature search or pre-submission review.*

Runs 5 layers per paper. Fetches the abstract from CrossRef raw JSON → Semantic Scholar → Unpaywall → arXiv → PubMed as fallback. Checks whether the abstract actually contains the claim you're citing it for.

---

## Example output

```
REFERENCE AUDIT
────────────────────────────────────────────────────────
Paper:   High-Speed Electrically Actuated Elastomers with Strain Greater Than 100%
DOI:     10.1126/science.287.5454.836 — ✓ Resolves (paywalled)
Authors: Ron Pelrine, Roy Kornbluh, Qibing Pei, Jose Joseph
Year:    2000 — Source: CrossRef + Semantic Scholar
Journal: Science, Vol. 287, Issue 5454, pp. 836-839

EXISTENCE:  ✓ Confirmed (CrossRef + S2, 2948 citing works)
METADATA:   ✓ Consistent
CONTENT:    ✓ Supported
            "Actuated strains up to 117% were demonstrated with silicone
            elastomers, and up to 215% with acrylic elastomers using biaxially
            and uniaxially prestrained films."
            [Source: CrossRef raw JSON abstract field]
RETRACTION: ✓ None found

VERDICT: ACCEPT
────────────────────────────────────────────────────────
```

```
REFERENCE AUDIT
────────────────────────────────────────────────────────
Paper:   Carpi et al. (2011) — Dielectric elastomers as electromechanical transducers
DOI:     10.1016/B978-0-08-047488-5.00001-0

EXISTENCE:  ✗ Mismatch — DOI resolves to Chapter 1 by Ronald Pelrine & Roy Kornbluh (2008),
            not Carpi et al.
METADATA:   ✗ Year: user provided 2011; CrossRef record: 2008
            ✗ Authors: "Carpi et al." are book editors; chapter authors are Pelrine & Kornbluh

VERDICT: REJECT
Reason: DOI resolves to a different paper. If citing the full edited book, use:
        Carpi, F. et al. (Eds.) (2008). DOI: 10.1016/b978-0-08-047488-5.x0001-9
────────────────────────────────────────────────────────
```

---

## How it works

Each paper goes through up to 5 layers:

1. **Existence** — CrossRef + Semantic Scholar independent search. Two-source confirmation required.
2. **Metadata** — Title, all authors, year, journal, DOI cross-checked across sources. Any mismatch shown explicitly.
3. **Content traceability** — Abstract fetched live (CrossRef JSON → S2 → Unpaywall → arXiv → PubMed). Every content statement in the output is quoted verbatim from the fetched text. If the abstract is inaccessible after all fallbacks, it is marked `UNVERIFIABLE` — never filled in from memory.
4. **DOI resolution** — `doi.org/{DOI}` fetched and landing page confirmed to match the expected paper.
5. **Retraction check** — Web search for retraction notices + DOI landing page banner check.

---

## The core rule

> Every content statement about a paper must come from a live-fetched abstract, quoted or paraphrased verbatim. If the abstract cannot be fetched, say so explicitly — never fill the gap with recalled description.

This is what separates `ref-verify` from asking Claude to "check" a citation. Without the skill, Claude pattern-matches against training data and describes what a paper "shows" with high confidence — even when wrong. With the skill, if Claude can't fetch the abstract, it says so instead of guessing.

---

## Near-miss detection

The skill evaluates the *specific claim* being cited, not just whether the paper exists and is about the right topic.

**Example:** Kofod et al. (2003) contains "500% strain" in the abstract. Without claim checking, this looks like strong support for "DEAs achieve >100% strain." With claim checking: the 500% is a pre-strain value at which breakdown field was measured — not an actuation output. The skill marks it `WARN (PARTIAL)` and explains why.

---

## Compatible with

- Claude Code (auto-triggers on citation tasks)
- Cursor, Codex, and other `npx skills` compatible agents

---

## Related

- [anneal-skill](https://github.com/moonweave/anneal-skill) — measure-first decision discipline
- [decide-skill](https://github.com/Moon-python/decide-skill) — decision automation for non-expert domains
