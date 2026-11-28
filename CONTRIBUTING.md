# Contributing to commit-scopes

This repository ships a GitHub composite action, not a wheel or application.
Its check reads commit subjects and workflow files that a pull request controls,
so tests must exercise the real script against real git repositories and place
fakes only at the network boundary.

## Local setup

Use Python 3.14 or 3.15. Bash and Git are required for the action-entrypoint
test; the fixtures are local repositories with a filesystem remote, so the tests
need no token and make no network call.

```bash
python3 -m venv .venv
. .venv/bin/activate
python -m pip install --require-hashes --only-binary=:all: -r requirements-test.txt
python -m pip install --require-hashes --only-binary=:all: -r requirements-lint.txt
```

On Windows, create the environment with `python -m venv .venv` and activate
`.venv\Scripts\Activate.ps1`. The action test uses Git Bash. The pinned
development tools (PyYAML, Ruff, and the `requirements-lint.txt` toolchain)
are development tools only; runtime modules must stay standard-library-only.

## Required checks

The `require-ci-aggregate` ruleset requires each CI context by name, as
claim's and pr-gate's do, so `tests.yml` has no aggregate job; no shell file
is tracked, so there is no shellcheck job.

Run these from the repository root with the virtual environment active:

```bash
python -m pytest -n auto tests/
python -m ruff check --select E9,F63,F7,F82 .
git ls-files "*.py" | xargs python -m pycodestyle
git ls-files "*.py" | xargs pylint --rcfile=.pylintrc
pyright --outputjson > /tmp/pyright-report.json
python scripts/ci/type_ratchet.py /tmp/pyright-report.json
actionlint -color .github/workflows/*.yml
zizmor --no-progress action.yml .github/workflows/
git diff --check
```

pytest runs every `tests/test_*.py` module; `tests/conftest.py` gives each test
its private `tmp` directory and fails the run on a module that collects no test
or on any skipped test. Individual suites run the same way, for example
`python -m pytest tests/test_commit_scopes.py`.
CI runs the same tests and Python lint on Linux, Windows, and macOS across both
supported Python versions. The separate workflow audit pins actionlint
1.7.12 and zizmor — zizmor's version lives in `requirements-zizmor.txt`,
hash-verified at install time and updated by Dependabot; its installation
steps checksum-verify actionlint.
Zizmor can run offline locally; disclose that limitation when reporting results.

The Python lint and type gates run the tools pinned in
`requirements-lint.txt`, and the suites run on `requirements-test.txt`; both
are installed with `--require-hashes --only-binary=:all:`, as the workflows
do, and Dependabot proposes the version bumps. pycodestyle's line ceiling is the
tree's measured convention — `setup.cfg` sets 100 columns — and admits E402
for the deliberate `sys.path`-before-import harness pattern. pylint's checked
in policy (`.pylintrc`) turns the style categories off; every per-message
disable in it names the real finding that forced it. The type gate is the
ratchet: pyright's error count is compared against `pyright-baseline.json`
and fails only when it rises. Fix the error instead of editing the baseline;
lower the baseline in the same commit as the fix it records. Never raise the
recorded number to admit a new error.

Use strict test-driven development: add a behavioral regression, run it RED for
the intended reason, implement the smallest fix, and run it GREEN. For gate
changes, plant a representative defect in the actual runtime/action file,
prove the control catches it, restore the correct code, and rerun the suite.
Enumerate the workflow `name:` shapes the reader refuses before changing it.

## Security boundaries

The action executes only code under its immutable `github.action_path`.
Never add a checkout of pull-request code, execute scripts from the consumer's
repository, or interpolate a commit subject or workflow text into shell source.
Commit subjects and workflow files are data.

Use pinned action references and explicit least-privilege permissions in this
repository's own CI. Add new code paths to the deny-by-default `.gitignore` and
prove they are visible with `git check-ignore -q` (exit 1) and `git status`.
Seed representative junk when reopening directories and prove it stays ignored.

## Issues, pull requests, and commits

Use the shipped [bug report](.github/ISSUE_TEMPLATE/bug-report.md) and
[PR template](.github/PULL_REQUEST_TEMPLATE.md). Keep required sections, remove
inapplicable optional/conditional sections and instructions, report exact
commands/results, and put closing references only in Related Issues and Pull
Requests. Bugs Discovered is a pointer list to already-filed issues with exact
titles, not a narrative findings section. The Footer is the final attribution line.

Every agent-authored commit carries the actual model's plain-name trailer:

```text
Co-Authored-By: GPT-6 Astra <noreply@openai.com>
```

Substitute the model that actually authored the work; omit effort or context
suffixes. Stage explicit paths, keep commits focused, and verify before
pushing. Commit subjects follow Conventional Commits (`type: summary`, with an
optional scope), and a workflow's `name:` under `.github/workflows/` is a
scope only with the `ci` type — `ci(claim)` changes the workflow, while any
other type on that scope is refused by `scripts/ci/commit_scopes.py`, which
derives the workflow-name set from the workflows themselves; take any other
scope from the file or area the commit is about.
Consumers pin the action by commit SHA.

Releases follow semantic versioning: a breaking commit bumps the major
version, a `feat` commit the minor, and anything else the patch. `v*` tags
are permanent reviewability metadata naming reviewed states; consumer pins
stay full SHAs with `# vX.Y.Z` version comments. Dependabot is expected to
propose moving the workflow's pin — SHA and version comment together — when
a release is cut.

Follow the [Code of Conduct](CODE_OF_CONDUCT.md). Enforcement reports go privately
to Nitjsefnie at [zmatek.peter@gmail.com](mailto:zmatek.peter@gmail.com).
