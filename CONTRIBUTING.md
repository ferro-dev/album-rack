# Contributing to album-rack

Thanks for your interest in `album-rack`. This document covers how to report
bugs, request features, and submit code.

> **Status: pre-development.** The project has just been scaffolded — there is
> no functional code yet. The most useful contribution right now is engaging
> with planning discussion in issues.

## Reporting bugs

Open a [bug report](https://github.com/ferro-dev/album-rack/issues/new?template=bug_report.md)
and fill in the template. A good report includes:

- `album-rack --version` output
- Your OS and version
- Exact command run and the full output (use a code block)
- What you expected versus what happened

## Requesting features

Open a [feature request](https://github.com/ferro-dev/album-rack/issues/new?template=feature_request.md).
Describe the problem you're trying to solve, not just the solution you have in
mind.

For open-ended ideas and questions, use
[Discussions](https://github.com/ferro-dev/album-rack/discussions) rather than
Issues.

## Development setup

`album-rack` is written in **Go 1.22+**.

```bash
git clone https://github.com/ferro-dev/album-rack.git
cd album-rack
go build ./...
go test ./...
```

### Code style

- `gofmt` (enforced; CI fails on unformatted code)
- `golangci-lint` (configuration in `.golangci.yml`) — install it with
  `go install github.com/golangci/golangci-lint/v2/cmd/golangci-lint@latest`
  or your distro's package.
- Enable the pre-commit hook with `make hooks` (equivalently
  `git config core.hooksPath .githooks`). It runs gofmt, `go vet`,
  `golangci-lint`, and the tests before each commit — the same gates CI
  enforces. `make check` runs them on demand.

## Submitting changes

1. Fork the repo and create a branch off `main`.
2. Make your change. **Tests are required** — code changes ship with
   corresponding test updates (add, update, or remove as appropriate).
3. Run `gofmt`, `golangci-lint`, and `go test ./...` locally; all must pass.
4. Open a PR and fill in the
   [pull request template](.github/pull_request_template.md).
5. `main` is protected — PRs require review before merge.

## Commit messages

Use [Conventional Commits](https://www.conventionalcommits.org/) style:
`feat:`, `fix:`, `docs:`, `chore:`, `refactor:`, `test:`. Keep the subject in
imperative mood and under 50 characters.

## License

By contributing, you agree that your contributions are licensed under the
[MIT License](LICENSE), the same license that covers the project.
