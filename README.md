# data-diff-next

> A modern, independently maintained foundation for cross-database table comparison.

`data-diff-next` is a new project inspired by the archived `data-diff` codebase. The project is
being designed around current Python packaging, typing, testing, and release practices while
keeping compatibility with established `data_diff` imports and the `data-diff` command wherever
that compatibility is deliberately implemented and tested.

> [!IMPORTANT]
> This repository is currently a project scaffold. The package metadata defines the distribution,
> optional dependencies, and console entry point, but the README does not claim application
> features that have not yet been implemented and tested.

## Project status

The current project foundation defines:

- distribution name: `data-diff-next`
- Python import package: `data_diff`
- console command: `data-diff`
- supported Python versions: 3.10 through 3.13
- build backend: Hatchling
- dependency and environment management: uv
- validation models: Pydantic 2
- linting and formatting: Ruff
- static type checking: Pyright
- testing: pytest

The following capabilities are architectural goals and must not be considered stable until their
implementations and public contracts are present in the repository:

- local and cross-database table comparison
- hash-diff and join-diff algorithms
- deterministic console, JSON, and JSONL reporting
- dbt and cloud integrations
- third-party database-driver plugins

See [`docs/roadmap.md`](docs/roadmap.md) for implementation status and planned milestones.

## Requirements

- Python 3.10, 3.11, 3.12, or 3.13
- [uv](https://docs.astral.sh/uv/) for the documented development workflow

Some optional database drivers require operating-system libraries or vendor client software. Refer
to the relevant driver's documentation before installing an extra in production.

## Installation

### Install from PyPI

The following commands apply after `data-diff-next` has been published to PyPI:

```bash
uv pip install data-diff-next
```

Standard pip is also supported by the package metadata:

```bash
python -m pip install data-diff-next
```

### Install from a local checkout

Clone the repository and synchronize the locked environment:

```bash
git clone https://github.com/thfroehlich/data-diff-next.git
cd data-diff-next
uv sync --all-groups
```

`uv sync` creates or updates the project environment and installs the local project as an editable
package. The committed `uv.lock` should be used for reproducible development and CI environments.

### Optional database drivers

Database drivers are published as optional extras rather than mandatory core dependencies.
Install only the drivers needed by the target environment.

```bash
uv pip install "data-diff-next[postgresql]"
uv pip install "data-diff-next[mysql]"
uv pip install "data-diff-next[mssql]"
uv pip install "data-diff-next[oracle]"
uv pip install "data-diff-next[duckdb]"
uv pip install "data-diff-next[clickhouse]"
uv pip install "data-diff-next[snowflake]"
uv pip install "data-diff-next[trino]"
uv pip install "data-diff-next[presto]"
uv pip install "data-diff-next[vertica]"
```

Install the aggregate database extra when all declared drivers are intentionally required:

```bash
uv pip install "data-diff-next[all-dbs]"
```

The `all-dbs` extra may include native drivers and should not be used by default in minimal
production environments.

### Optional integrations

Install dbt support:

```bash
uv pip install "data-diff-next[dbt]"
```

Install optional cloud-integration dependencies:

```bash
uv pip install "data-diff-next[cloud]"
```

## Command-line interface

The project metadata registers this console script:

```toml
[project.scripts]
data-diff = "data_diff.cli.main:main"
```

After installation, verify that the entry point can be loaded:

```bash
data-diff --help
```

The CLI's command syntax and supported options are defined by `data_diff.cli.main:main`. Until that
module and its CLI contract tests exist, this README intentionally does not document comparison
arguments or options such as `--format`, `--json`, database URLs, or table parameters.

When module execution is implemented through `data_diff/__main__.py`, it can additionally be
verified with:

```bash
python -m data_diff --help
```

Do not treat `python -m data_diff` as available solely because the console script is declared in
`pyproject.toml`; module execution requires a real `data_diff/__main__.py` implementation.

## Python API

The import package declared by the project is `data_diff`:

```python
import data_diff
```

No table-diff function, service object, result model, or call signature is documented here until a
public implementation exists under `data_diff.api` and is protected by compatibility tests. In
particular, examples using an unverified function such as the following are intentionally omitted:

```python
# Not a supported example until this symbol is implemented and tested:
# from data_diff.api import diff_tables
```

The eventual public API contract will be documented in
[`docs/public-api.md`](docs/public-api.md). New public APIs must include type annotations, tests,
and migration notes when they replace an established interface.

## Development

### Install uv

Follow the official uv installation instructions for the development platform, then verify the
installation:

```bash
uv --version
```

### Synchronize the environment

Install the project and every declared development dependency group:

```bash
uv sync --all-groups
```

Check that the lockfile matches `pyproject.toml` without updating it:

```bash
uv lock --check
```

### Run tests

Run the default test suite:

```bash
uv run pytest
```

Run a marker-specific subset:

```bash
uv run pytest -m unit
uv run pytest -m integration
uv run pytest -m database
uv run pytest -m network
uv run pytest -m slow
```

The default test run should not require cloud credentials or external database services. Tests that
require such resources must use the appropriate marker and document their environment variables.

### Lint and format

Run Ruff linting:

```bash
uv run ruff check .
```

Check formatting without modifying files:

```bash
uv run ruff format --check .
```

Apply deterministic formatting locally:

```bash
uv run ruff format .
```

### Type checking

Run Pyright:

```bash
uv run pyright
```

### Build distributions

Build the wheel and source distribution:

```bash
uv build
```

The generated artifacts are written to `dist/`.

### Validate the built wheel

On macOS or Linux:

```bash
python -m venv .wheel-test
. .wheel-test/bin/activate
python -m pip install dist/*.whl
python -c "import data_diff"
data-diff --help
```

On Windows PowerShell:

```powershell
py -m venv .wheel-test
.\.wheel-test\Scripts\Activate.ps1
python -m pip install (Get-ChildItem dist\*.whl | Select-Object -First 1)
python -c "import data_diff"
data-diff --help
```

Remove the temporary environment after validation:

```powershell
deactivate
Remove-Item -Recurse -Force .wheel-test
```

### Run the complete local quality gate

```bash
uv lock --check
uv sync --all-groups
uv run ruff check .
uv run ruff format --check .
uv run pyright
uv run pytest
uv build
```

## Dependency model

The project uses three dependency categories:

- `[project.dependencies]` for runtime dependencies installed for every user
- `[project.optional-dependencies]` for published feature and database extras
- `[dependency-groups]` for local development tools such as pytest, Ruff, Pyright, and documentation tooling

Pydantic is constrained to major version 2 by the project metadata. Database drivers remain optional
to keep the core installation small and avoid unnecessary native dependencies.

## Repository layout

```text
data_diff/
├── api/
├── cli/
├── compatibility/
├── config/
├── core/
│   ├── algorithms/
│   ├── execution/
│   ├── models/
│   └── results/
├── databases/
│   ├── base/
│   ├── dialects/
│   └── drivers/
├── integrations/
│   ├── cloud/
│   └── dbt/
├── plugins/
├── reporting/
└── telemetry/
```

This is an architectural target, not evidence that every package already contains a stable public
implementation.

## Architecture rules

- Core algorithms must not import Click, Rich, cloud clients, or database presentation code.
- CLI commands must call the same public service layer used by Python callers.
- Database drivers must not format terminal output.
- Reporting components consume structured result objects.
- Configuration models must not open database connections.
- Cloud, dbt, telemetry, and database-driver dependencies are optional and lazy-loaded where practical.
- Telemetry is disabled by default and must never include credentials, connection strings, SQL values,
  or compared row data.

Architecture decisions are recorded in [`docs/adr/`](docs/adr/).

## Documentation

- [Vision](docs/vision.md)
- [Roadmap](docs/roadmap.md)
- [Architecture](docs/architecture.md)
- [Public API](docs/public-api.md)
- [Output formats](docs/output-formats.md)
- [Migration guide](docs/migration-guide.md)
- [Dependency policy](docs/dependency-policy.md)

Links should be removed or marked as planned if the referenced files are not yet committed.

## Contributing

Read [`CONTRIBUTING.md`](CONTRIBUTING.md) before opening a pull request. Contributions should:

- include tests for behavior changes
- preserve compatibility unless a breaking change is documented
- update documentation for public behavior
- add or update an ADR for significant architectural decisions
- pass lint, format, type, test, and build checks

## Security

Report vulnerabilities using the private process documented in [`SECURITY.md`](SECURITY.md). Do not
open public issues containing credentials, connection strings, exploit details, or sensitive data.

## Project lineage and independence

`data-diff-next` is an independent successor project inspired by the archived `data-diff` project.
It is not affiliated with, endorsed by, or maintained by the original upstream maintainers. Any
compatibility claims must be backed by tests and documented migration guidance.

## License

This project is licensed under the MIT License. See [`LICENSE`](LICENSE).
