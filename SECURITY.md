# Security Policy

## Supported versions

`album-rack` is pre-1.0 and pre-release. Until a `0.1.0` tag exists, only the
`main` branch is supported. Once releases begin, this section will list which
versions receive security fixes.

## Reporting a vulnerability

**Do not open a public issue for security vulnerabilities.**

Use GitHub's private vulnerability reporting:

1. Go to the [Security tab](https://github.com/ferro-dev/album-rack/security/advisories).
2. Click **Report a vulnerability**.
3. Provide a description, reproduction steps, affected version/commit, and
   impact.

You can expect an initial acknowledgement within a few days. Once the report is
triaged, we will coordinate a fix and a disclosure timeline with you.

## Scope

Relevant concerns for a local album-tracking CLI include, but are not limited
to:

- Path traversal when reading or writing library data files.
- Unsafe handling of untrusted metadata from any remote sources added later
  (e.g. MusicBrainz, Discogs).
- Injection via imported/exported data (CSV, JSON, etc.).

## Out of scope

- Issues that require an already-compromised local account.
