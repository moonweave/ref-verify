# Release Automation Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add CI, manual live smoke, PyPI publishing guardrails, and package smoke verification.

**Architecture:** GitHub Actions owns remote automation. A local Python script
owns package artifact checks so CI and maintainers can run the same validation.
Docs explain the workflows without changing runtime behavior.

**Tech Stack:** GitHub Actions, Python stdlib, `build`, `twine`, `pypa/gh-action-pypi-publish`.

---

### Task 1: Package Smoke Script

**Files:**
- Create: `scripts/package_smoke.py`

- [ ] Create a script that accepts `--wheel` and `--expected-version`.
- [ ] Create a temporary virtualenv.
- [ ] Install the wheel into that virtualenv.
- [ ] Run `ref-verify --help` and `ref-verify check-claim --help`.
- [ ] Import `ref_verify.__version__` and compare it to the expected version.
- [ ] Inspect the wheel and fail if `SKILL.md` is packaged.

### Task 2: GitHub Actions Workflows

**Files:**
- Create: `.github/workflows/ci.yml`
- Create: `.github/workflows/live-smoke.yml`
- Create: `.github/workflows/publish-pypi.yml`

- [ ] CI runs on push and pull request with Python 3.10 and 3.12.
- [ ] CI runs unit tests, byte compilation, build, twine check, and package smoke.
- [ ] Live smoke is manual-only and runs representative DOI checks.
- [ ] PyPI publishing runs only on published `v*` GitHub releases and uses OIDC trusted publishing.

### Task 3: Contributor Documentation

**Files:**
- Modify: `CONTRIBUTING.md`
- Modify: `README.md`
- Modify: `README.ko.md`

- [ ] Document local verification commands.
- [ ] Explain that live smoke is manual because it hits public APIs.
- [ ] Explain that PyPI publishing requires trusted publisher setup.

### Task 4: Verification

- [ ] Run `PYTHONPATH=src python3 -m unittest discover -s tests -v`.
- [ ] Run `python3 -m py_compile src/ref_verify/*.py tests/*.py scripts/*.py`.
- [ ] Build wheel and sdist.
- [ ] Run `twine check`.
- [ ] Run `python3 scripts/package_smoke.py --wheel <built wheel> --expected-version 1.1.1`.
- [ ] Run `git diff --check`.

