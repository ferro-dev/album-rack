# album-rack — Project & Implementation Plan

## 1. Overview

`album-rack` is a cross-platform command-line tool for tracking a physical
(and digital) music collection: what albums are owned, in which format(s),
and as which specific edition/pressing. It solves the problem of album
editions (deluxe editions, remasters, live releases, reissues) getting
tracked as disconnected, unrelated entries by modeling each one as a
release under a shared album.

Runs on Linux, macOS, and Windows 10+. Data lives in a local SQLite
database with an explicit flat-file export for backup and version control.

## 2. Goals

- Fast, low-friction entry of owned albums, in two modes:
  - **Quick entry**: minimal required fields, immediate and specific
    validation errors on anything missing.
  - **Guided/wizard entry**: step-by-step form covering the full field
    set, for careful cataloging.
- Detect likely duplicates and typos on entry via fuzzy name matching,
  and let the user resolve the match as another release, a genuinely new
  album, or abort the entry.
- Browse and search the collection with multiple sort orders.
- Store data locally, with a portable, backup-friendly, git-trackable
  export format.
- Run identically on Linux, macOS, and Windows 10+ as a single
  self-contained binary with no external runtime dependencies.
- Ship with tests as a required part of every change, not an afterthought.

## 3. Non-Goals (MVP)

- No network calls, no online metadata lookup or validation.
- No GUI. Interface is a CLI + terminal UI (TUI).
- No multi-user, sync, or cloud storage.
- No market-price tracking or marketplace integration.
- No vinyl sub-format detail (speed, size, color), country of release, or
  genre/style fields — reserved for a later iteration if useful.

Deferred but designed to not be architecturally precluded: online lookup
and validation against external metadata sources, cross-referencing a
digital library, and market-price tracking. See §9.

## 4. Data Model

Two entities, in a one-to-many relationship: an **Album** is the abstract
work; each **Release** is one specific owned edition-in-a-format of that
album (the term follows Discogs/MusicBrainz usage, where a "release" is a
specific pressing/edition, distinct from the work it belongs to). A user
who owns the same album on CD and vinyl has one Album and two Releases. A
deluxe reissue is a third Release under the same Album, not a separate
Album.

### Album

| Field | Required | Notes |
|---|---|---|
| `artist` | yes | Album artist. Used (with `name`) for fuzzy match on entry. |
| `name` | yes | Album title. |
| `original_release_year` | no | The work's original release year, independent of any specific pressing. |
| `created_at` / `updated_at` | system | Timestamps. |

### Release

| Field | Required | Notes |
|---|---|---|
| `album_id` | yes | Parent album. |
| `format` | yes | One of: `vinyl`, `cd`, `cassette`, `digital`, `other`. Exactly one per release — owning the same edition in two formats is two releases. |
| `edition` | no | Free text: `Deluxe Edition`, `2019 Remaster`, `Live`, etc. Blank means standard/original edition. |
| `release_date` | no | This specific release/pressing's date (distinct from the album's original release year). |
| `label` | no | Record label for this release. |
| `featured_artists` | no | Additional credited artist(s) for this release. |
| `catalog_number` | no | Label's catalog number for this release. |
| `barcode` | no | UPC/EAN barcode. |
| `media_condition` | no | Goldmine scale: Mint, Near Mint, VG+, VG, G+, G, Fair, Poor. |
| `sleeve_condition` | no | Same scale, applied to sleeve/packaging separately. |
| `purchase_date` | no | When the user got this copy. |
| `purchase_price` | no | What the user paid. |
| `purchase_source` | no | Where/who it was acquired from. |
| `storage_location` | no | Free text for physical storage location. |
| `notes` | no | Free text. |
| `discogs_release_id` / `musicbrainz_release_id` | no | Reserved, unused until online lookup ships. Stored so future integration doesn't need to re-match existing entries — these map directly onto this same Release entity on both external services. |
| `created_at` / `updated_at` | system | Timestamps. |

## 5. Entry Flows

### Quick entry

A single command accepting the required fields (artist, name, format)
plus any optional fields as flags. Missing required fields produce an
immediate error naming every missing field at once (not one at a time).
On success, runs duplicate/release detection (§6) before committing.

### Wizard entry

A guided, multi-step terminal form covering the full field set in logical
groups (album identity; format and edition; physical/condition details;
purchase details), with optional groups skippable. Runs the same
duplicate/release detection as quick entry, at the point the album
identity fields are confirmed, before the rest of the wizard proceeds.

### Duplicate / Release Detection

On every new entry, normalize the entered artist and album name
(lowercase, trim, strip punctuation) and compare against existing albums
using Jaro-Winkler similarity. Similarity above a fixed threshold surfaces
a prompt showing the matched album(s), with three actions:

1. **Another release of this album** — create the new entry as a Release
   under the matched Album.
2. **New album** — create a new Album (the match was a false positive).
3. **Abort** — discard the entry entirely.

The threshold is fixed in code, not user-configurable, and tuned to favor
catching real duplicates/typos over minimizing prompts.

## 6. Browse & Search

List and filter the collection by artist, format, and edition. Results
are sortable by:

- Relevance (only meaningful alongside a search term)
- Album artist
- Original release year
- Date added

Browsing is available both as a scriptable CLI command (for quick lookups
and scripting) and as an interactive terminal view (for casual browsing).

## 7. Storage & Backup

SQLite is the canonical working store, kept in the OS-appropriate user
config/data directory. Two additional commands make the data portable:

- **Export**: writes the full collection to a flat, human-readable file
  (one file per album, YAML), suitable for committing to a git
  repository.
- **Import**: rebuilds the SQLite store from an exported flat-file tree —
  used to restore on a new machine, or to reconcile after pulling changes
  from git.

The flat-file export is the durable backup artifact; the SQLite database
is a regenerable working cache.

## 8. Architecture & Tech Stack

- **Language**: Go (1.22+ floor).
- **CLI structure**: Cobra-based command tree (`add`, `list`, `search`,
  `export`, `import`, etc.), consistent quick-entry-via-flags and
  wizard-via-subcommand-or-flag on the same underlying data layer.
- **Terminal UI**: Bubble Tea for interactive views (browse), `huh` for
  guided/wizard forms — both produce a single static binary per OS with
  no runtime dependencies.
- **Storage**: pure-Go SQLite driver (no cgo), so cross-compilation for
  all three target OSes stays simple.
- **Flat-file format**: YAML, one file per album.
- **Fuzzy matching**: Jaro-Winkler similarity over normalized strings.

## 9. Future / Deferred Work

Explicitly out of MVP scope but not architecturally blocked:

- Online metadata lookup and validation (e.g. against public music
  databases) to autofill or verify entries.
- Cross-referencing a digital music library maintained by a separate
  tool, to unify physical and digital ownership tracking.
- Market-price / listing monitoring for owned or wanted albums.
- Additional optional fields: vinyl-specific attributes (speed, size,
  color), country of release, genre/style.
- A graphical (non-terminal) interface, if the TUI proves limiting for
  some users.

## 10. Testing Strategy

Testing is a required part of every change, not a separate phase:

- **Unit tests** for all data-model, validation, and fuzzy-matching logic
  — written as pure functions independent of the TUI/CLI layer so they're
  fast and simple to test in isolation.
- **Integration tests** against a real (temporary, per-test) SQLite
  database for storage, CRUD, and query/sort behavior.
- **Export/import round-trip tests** to guarantee the backup path is
  reliable: export, wipe, import, verify identical data.
- **CLI-level tests** invoking the built binary against a temporary
  database for command surface behavior (quick entry validation errors,
  duplicate-detection prompt flow, list/search/sort output).
- Table-driven test style throughout, matching standard Go convention.
- Continuous integration runs the full suite (with the race detector) on
  every push and pull request, across all three target platforms, and
  must be green before merge.

## 11. Distribution

Cross-compiled release binaries for Linux, macOS, and Windows (amd64 and
arm64), built and published automatically from tagged releases. Each
release ships as a plain binary per platform plus checksums — no
installer or package-manager distribution in the MVP.

## 12. Roadmap

- **M0 — Repository bootstrap**: module setup, CI, dev tooling, community
  docs. *(Complete.)*
- **M1 — Data model & storage**: SQLite schema, Album/Release structs,
  CRUD operations, unit and integration tests.
- **M2 — Quick entry**: command, required-field validation and error
  reporting, tests.
- **M3 — Duplicate/release detection**: normalization, similarity
  matching, resolution prompt flow, tests.
- **M4 — Wizard entry**: guided multi-step form covering the full field
  set, tests.
- **M5 — Browse & search**: filtering, sorting, CLI and interactive
  views, tests.
- **M6 — Flat-file export/import**: backup/restore round trip, tests.
- **M7 — Packaging & first release**: cross-platform build pipeline,
  verified binaries for all three OSes, tagged `v0.1.0`.
- **Post-1.0**: items in §9, prioritized based on real usage.
