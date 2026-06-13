# AGENTS.md

This file provides guidance to AI agents working in this repository.
`charm-ubuntu` is the Juju charm that deploys a blank Ubuntu cloud image (a
testing/development charm with no application payload). The charm code is a
single file, `src/charm.py`, built on `ops`.

## Lint, test

Everything runs through `tox`:

```bash
tox -e lint         # flake8 + black --check (this charm uses black, not ruff)
tox -e unit         # unit tests under tests/unit
tox -e integration  # deploys to LXD; needs juju and charmcraft (packs the charm)
```

The integration env expects a Juju controller on LXD and `charmcraft` (the
channel is pinned in `.charmcraft-channel`); it packs and deploys the charm,
so it can't run without that environment.

## Conventions

- **`black` and `flake8`, not `ruff`** — line length 88. Tests set
  `PYTHONPATH=src` so `import charm` resolves.
- **Wide Python support:** CI lints and unit-tests on 3.6, 3.8, 3.10, and 3.12.
  Keep `src/charm.py` compatible with 3.6.
- **`ops` is pinned to `>=1.0,<2.0`** (`requirements.txt`) — this charm tracks
  the 1.x API, not current `ops`.
- **Commits / PR titles:** [Conventional Commits](https://www.conventionalcommits.org/)
  (`chore`, `ci`, `docs`, `feat`, `fix`, `perf`, `refactor`, `revert`, `test`),
  no scopes.
