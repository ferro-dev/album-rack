# Changelog

All notable changes to `album-rack` are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- `LICENSE` (MIT) and `README.md`.
- Go module (`go.mod`, Go 1.22+) and a minimal CLI entrypoint.
- Community infrastructure: `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`,
  `SECURITY.md`, `CHANGELOG.md`, GitHub issue and pull request templates.
- Continuous integration (GitHub Actions): gofmt + `go vet`, golangci-lint,
  and build + test across Ubuntu 22.04/24.04, all on the Go 1.22 floor.
- Linting and dev tooling: `golangci-lint` config (`.golangci.yml`) plus a CI
  lint job, a `.githooks/pre-commit` hook mirroring the CI gates, and a
  `Makefile` (`build`/`test`/`vet`/`fmt`/`lint`/`hooks`/`check`).

[Unreleased]: https://github.com/ferro-dev/album-rack/commits/main
