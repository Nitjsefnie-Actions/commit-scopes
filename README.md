# commit-scopes

Fail a pull request commit whose scope names a workflow while its type is not
`ci`.

A repository's CONTRIBUTING can allow a workflow's `name:` as a commit scope
only with the `ci` type: `ci(claim)` changes the workflow, while `fix(claim)`
would claim to fix some other file. This action reads that rule instead of
leaving it as prose.

## Usage

Run it in a job after the checkout, on the runner's `python3` (standard
library only). The action fetches the repository's default branch itself.

```yaml
      - uses: actions/checkout@3d3c42e5aac5ba805825da76410c181273ba90b1 # v7.0.1
        with:
          persist-credentials: false
      - uses: Nitjsefnie-Actions/commit-scopes@645ca8f43b00199cebeed7a5d305b4224e902a60 # v1.0.0
```

Put the step in a job the required aggregate already folds: a new job is a new
required context only once the ruleset or the aggregate names it. The action
has no inputs.

## What it checks

- The workflow-name set is derived at `HEAD` from the tracked files under
  `.github/workflows`: the top-level `name:` of each one, never its filename
  and never a job's or step's `name:`.
- Only the outgoing range `origin/<default branch>..HEAD` is examined, and the
  green line says so. Merged history is never re-judged.
- A commit whose subject is `type(scope): summary`, with a scope that equals a
  workflow name exactly, fails unless its type is `ci`. A `!` after the type or
  the scope is read.
- A subject that does not parse (a merge commit, a non-conforming subject) is
  counted and listed, never failed.
- A workflow `name:` the reader cannot read (quoted, anchored, a block scalar,
  with a trailing comment, continued onto the next line, or missing) is a
  refusal naming the file: an incomplete set would be a green over a gate that
  was never checked.

## Development and community

See [CONTRIBUTING.md](CONTRIBUTING.md), [SECURITY.md](SECURITY.md) and the
[Code of Conduct](CODE_OF_CONDUCT.md).
