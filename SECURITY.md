# Security

Report vulnerabilities privately through a
[security advisory on this repository](https://github.com/Nitjsefnie-Actions/commit-scopes/security/advisories/new).
Include the action commit SHA, the calling workflow and the observed behavior
needed to reproduce the problem. Use a repository you control and keep
sensitive details out of public issues.

The action and this repository's workflows are in scope. The action reads
commit subjects and workflow files that a pull request's author controls, so
both are untrusted data: they are passed to git as separate arguments and
never become shell or Python source. The action executes only code under its
immutable `github.action_path` and never runs code from the checked-out
repository.
