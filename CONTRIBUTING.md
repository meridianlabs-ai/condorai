# Contributing to Condor AI

## Development setup

Condor AI targets Python 3.10+ and needs [uv](https://docs.astral.sh/uv/) 0.13.4 or later. Clone the repository and install it in editable mode with the `dev` optional dependencies:

```bash
git clone https://github.com/meridianlabs-ai/condorai
cd condorai
uv sync --extra dev
```

`uv sync` creates `.venv` with the exact versions in `uv.lock`, the `dev` extra included, and installs Condor AI editable. Run Python commands from it: `uv run <command>`, `.venv/bin/<command>`, or `source .venv/bin/activate`. `uv sync` without `--extra dev` removes the dev tools again.

### Dependencies and the cooldown

`uv.lock` pins every Python dependency except the build toolchain. CI's lint, type-check, test and build jobs install their dependencies from it, and fail when `uv.lock` is out of date with `pyproject.toml`. CI's build job and the release build use the build toolchain (build, hatchling, hatch-vcs) pinned with hashes in `scripts/build-dist/requirements.txt`. Dependabot updates that file weekly, and `.github/workflows/publish.yaml` gives the command to update it by hand. For the editable install in the other jobs and on your machine, uv gets hatchling and hatch-vcs from PyPI, with the seven-day cooldown below. When you change dependencies in `pyproject.toml`, run `uv lock` and commit both files. When you change the `hatchling` requirement in `[build-system]`, make the same change in `scripts/build-dist/requirements.in` and compile `requirements.txt` again.

`pyproject.toml` sets `exclude-newer = "7 days"` under `[tool.uv]`, so `uv lock`, `uv sync` and `uv pip` only pick releases that have been on PyPI for at least seven days. A compromised release is usually found and removed within that time, before it reaches `uv.lock`, CI or your machine. A few packages wait 6 hours instead, so Condor AI can adopt a new release the day it ships: the `anthropic` and `openai` SDKs. They are listed with `"6 hours"` in `[tool.uv.exclude-newer-package]`. Only a direct dependency (a package in `dependencies` or the `dev` extra in `pyproject.toml`) that is a model provider SDK or an Inspect package gets this exception. Every other package keeps the seven-day cooldown. Dependabot's `uv` entry for `uv.lock` (directory `/`) in `.github/dependabot.yml` repeats the cooldown and the list. When you add a `"6 hours"` entry, also add the package to `cooldown.exclude` and the `six-hour-cooldown` group there. `tests/test_dependency_cooldown.py` checks that each entry is a direct dependency, that dated entries (below) are less than seven days old, and that the two files match. Whether a package is a provider SDK or an Inspect package is up to review.

Dependabot proposes `uv.lock` updates daily, once a release has cleared its cooldown. To adopt a release sooner, run `uv lock --upgrade-package <package>` and commit `uv.lock`. A release still inside its cooldown can't be locked; wait for it rather than overriding `exclude-newer`, unless it's an emergency (below).

In an emergency, for example a security fix, you can adopt a release that is less than seven days old with a one-time dated entry:

1. Under `[tool.uv.exclude-newer-package]` in `pyproject.toml`, add `<package> = "<timestamp>"`. The package must be a direct dependency. Use an RFC 3339 timestamp just after the upload time of the fixed release. Don't add a dated entry to `.github/dependabot.yml`.
2. Run `uv lock --upgrade-package <package>` and commit both files.
3. Within seven days, once the normal cooldown includes the release, delete the entry, run `uv lock --upgrade-package <package>`, and commit both files. A plain `uv lock` doesn't reliably remove the entry from `uv.lock`, and CI then fails. A test fails when a dated entry is seven days old or older.

## Checks and tests

Run the commands below in the activated `.venv`, or prefix them with `uv run`:

```bash
make check
make test
```
