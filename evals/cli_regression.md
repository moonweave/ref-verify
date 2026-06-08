# CLI regression corpus (ship-gate)

`cli_regression.jsonl` is a labeled, deterministic regression set for the
`check-file` engine. It complements `evals.json` (which evaluates skill-level LLM
behavior); this one pins **machine-checkable verdicts** so unit/source/matcher
changes can be regression-tested without an LLM in the loop.

Each row carries the claim **plus** ground-truth labels:

| field | meaning |
|---|---|
| `expected_verdict` | the verdict the engine *should* reach |
| `must_accept` | invariant: this row must end `ACCEPT` on every commit |
| `must_not_accept` | invariant: this row must **never** end `ACCEPT` |
| `gated_on` | open issues that currently block `expected_verdict` |
| `reachable_via` | where an abstract exists: `crossref` / `openalex` / `none` |
| `category` | `numeric_supported`, `fabricated_control`, `relational_out_of_scope`, `unreachable_ceiling`, `dead_doi_control`, `over_acceptance_regression` |

## Two invariant classes

**SAFETY (release blocker).** `must_accept` rows must stay `ACCEPT`; `must_not_accept`
rows must never become `ACCEPT`. This is the tool's core promise — no fabricated,
relational, unreachable, or over-accepting claim is waved through, and the one
clean supported claim stays green. A break here fails the gate (non-zero exit).

**PROGRESS (informational).** Gated rows do not yet reach `expected_verdict`
because a fix has not landed. They are reported, not failed, and flip to PASS as
their `gated_on` issue is resolved. This is how the corpus tracks the roadmap.

## How to run

```bash
PYTHONPATH=src python3 evals/run_cli_regression.py
```

Exit code is non-zero iff a SAFETY invariant is violated. (Live network: CrossRef /
OpenAlex / Semantic Scholar / PubMed. Semantic-Scholar free-tier 429 only affects
PROGRESS rows that depend on it, never SAFETY rows.)

## What the corpus encodes (snapshot, latest `main`)

```
SAFETY: 12/12 ok  |  PROGRESS pending: 3
Pending: A2-bellucci-30C, B2-diez-200g, E2-pelrine-117
```

- **Supported happy paths** — `A1` (`>220 °C` entails `>200 °C`) and `A3` (`1.7 eV`,
  unblocked once #14 landed) ACCEPT and must stay green.
- **Never-accept controls (all PASS)** — `B1` fabricated number, `C1`/`C2` relational,
  `D1`/`D2` unreachable (Elsevier / old Nature, abstract-only ceiling), `E1` dead DOI.
  `B3` (over-acceptance) is now `PARTIAL` after #11 — kept `must_not_accept` so the bug
  cannot silently regress.
- **Remaining gated false-negatives** (target ACCEPT once fixed): `B2` → #10,
  `E2` → up-to comparator, `A2` → residual condition handling (#13 already makes it
  reachable via OpenAlex, but a trailing `in the range` qualifier still blocks ACCEPT).

The verdict labels for `A2`/`A3`/`B2` were grounded by fetching the live abstracts
(CrossRef + OpenAlex) and confirming the value appears verbatim; no label asserts
support that is not in a fetched abstract.
