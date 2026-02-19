# Contributing

All contributions are welcome, such that updates for documentation, new code,
assistance with issue triage, issues for bug reports or feature requests,
feedback discussions, ... in respect to the Pylint [code of conduct][].

## Code checking

The project currently uses and requires [ruff](https://docs.astral.sh/ruff/)
for formatting and basic linting, along with [mypy](https://www.mypy-lang.org/)
for type checking. It is fully type checked, enforced by configuration.

The test suite is located in `tests/` and uses
[pytest](https://docs.pytest.org/en/stable/). A
[tox](https://github.com/tox-dev/tox) configuration is available to facilitate
running all the checkers locally, but `uv run` can be used to run the tools
explicitly.

[contributor guides]:
    https://pylint.readthedocs.io/en/stable/development_guide/contributor_guide/
[code of conduct]:
    https://github.com/pylint-dev/pylint/blob/main/CODE_OF_CONDUCT.md