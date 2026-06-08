# Release Automation Design

## Goal

Make `ref-verify` safer for non-developer users by automatically checking that
source tests, package builds, wheel installs, and release publishing paths stay
consistent.

## Scope

- Add a normal CI workflow for pushes and pull requests.
- Add a manual live-smoke workflow for public academic API checks.
- Add a GitHub Release-triggered PyPI publishing workflow using trusted
  publishing.
- Add a local packaging smoke script that installs a built wheel, checks the
  `ref-verify` console script, confirms the package version, and confirms the
  Python wheel remains CLI-only.
- Document the release checks in contributor-facing docs.

## Non-Goals

- Do not publish to PyPI in this change.
- Do not add live API tests to default CI.
- Do not add new runtime dependencies.
- Do not package `SKILL.md` into the Python wheel.

## Design

Default CI uses Python 3.10 and 3.12. It runs unit tests from the source
checkout, byte-compiles Python files, builds wheel and sdist artifacts, checks
metadata with `twine`, and runs a wheel install smoke test. This catches the
main failure modes that matter for users: broken tests, broken package metadata,
missing console scripts, and accidental skill-file packaging drift.

The live-smoke workflow is manual-only because CrossRef, Semantic Scholar, and
PubMed availability can be flaky. It verifies representative public-API CLI
paths without making default CI dependent on external services.

The PyPI workflow runs only when a GitHub Release is published for a `v*` tag.
It uses GitHub OIDC trusted publishing through `pypa/gh-action-pypi-publish`,
so no long-lived PyPI token needs to be stored in repository secrets.

## Success Criteria

- `python3 scripts/package_smoke.py --wheel <wheel> --expected-version 1.1.1`
  passes after a local build.
- GitHub Actions has workflows for CI, manual live smoke, and release publishing.
- README/CONTRIBUTING tell maintainers how these checks relate to installation
  and release safety.

