Welcome to Condor AI.

## Installation

Latest development version:

```bash
pip install git+https://github.com/meridianlabs-ai/condorai
```

## Development

To work on development of Condor AI, clone the repository and install it with [uv](https://docs.astral.sh/uv/) (0.13.4 or later) and the `[dev]` optional dependencies:

```bash
git clone https://github.com/meridianlabs-ai/condorai
cd condorai
uv sync --extra dev
```

Run the Python commands below from the `.venv` that `uv sync` creates: `uv run <command>`, `.venv/bin/<command>`, or `source .venv/bin/activate`. Before you change dependencies, read [CONTRIBUTING.md](CONTRIBUTING.md).

Run linting, formatting, and tests via

```bash
make check
make test
```


