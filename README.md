# recall-io

> PRD-agentic-memory §A item 11 (recall export / recall import) — round-trippable serialization of the v0.1 recall store as NDJSON.

## Install

### One-liner

```sh
curl -fsSL https://raw.githubusercontent.com/j0yen/recall-io/main/install.sh | bash
```

### Manual

```sh
git clone --depth 1 https://github.com/j0yen/recall-io.git
cd recall-io
./install.sh
```

Installs the `recall-io` binary via `cargo install --path . --locked`. Requires `cargo` / `rustc 1.85+` and `git`. Built binary lands in `~/.cargo/bin/`.

## Why

PRD-agentic-memory §A item 11 (recall export / recall import) — round-trippable serialization of the v0.1 recall store as NDJSON. Closes the remaining standalone-buildable v0.2 item alongside recall-doctor (slice 1) and recall-ops (slice 2). Use cases: backup/restore, migration to v0.2 schema, manual editing in a single editable file.

## Build

```sh
cargo build --release
```

Produces `target/release/recall-io`. Symlink into `~/.local/bin/` if you want it on `$PATH`.

## Usage

```sh
recall-io --help
```

## Audience

the author backing up `~/.claude/recall/` to a single file for safekeeping, restoring after `~/.claude/` corruption, or migrating between machines. Output: NDJSON to stdout (one memory per line). Input for import: a previously-exported NDJSON file (or any conforming stream).

## Acceptance criteria

This project was scaffolded from a PRD via the `autobuilder` pipeline. The MUST-level acceptance criteria are:

- **AC1**: `recall-io export [--root <dir>]` emits one JSON object per memory to stdout (NDJSON, one per line). Each object contains: `id`, `kind`, `subject`, `path`, `confidence`, `created_at`, `last_recalled_at`, `recall_count`, `embedding_id`, `...
- **AC2**: Export is deterministic: sorted by id ascending. Two consecutive exports of the same store produce byte-identical output.
- **AC3**: `recall-io import <jsonl-path>` reads the NDJSON stream, writes one .md file per memory to `<root>/memories/<subject>/<id>.md` (with frontmatter + body), and INSERTs rows into memories_meta. Creates the root dir + index if absent.
- **AC4**: Import is additive by default — duplicate ids (already present in memories_meta) are SKIPPED with a stderr warning, original row + file untouched. Reports `{imported: N, skipped: M, errors: K}` in JSON mode.
- **AC5**: `--replace` flag overrides skip behavior: duplicate ids cause the existing .md + index row to be replaced with the imported version.
- **AC6**: Round-trip: export a fixture store → wipe → import the export → all ids present in memories_meta, file count matches, body content preserved byte-identically.
- **AC7**: Malformed JSON lines on import → log to stderr with line number, count in `errors`, continue. Lines missing required fields (id, kind, subject) → same treatment.
- **AC8**: Exit codes: 0 = success; 1 = expected failure (no candidates exported on empty store with --strict; all import lines invalid); 2 = invocation error (bad path, sqlite3 unavailable, --root that's not a dir).
- **AC9**: `--format json` for import emits a single summary JSON object to stdout; text mode emits a one-line summary plus per-line warnings to stderr.

Each AC has a matching integration test under `tests/acceptance_ac<n>.rs`.

## Provenance

Built via the [`autobuilder`](https://github.com/j0yen/autobuilder) pipeline (PRD intake -> intent-card -> scaffold -> iterate-and-prove). Originally consolidated as a subdir of the [`wintermute`](https://github.com/j0yen/wintermute) monorepo; this standalone repo is a fresh-init snapshot for easier consumption and distribution.

## License

Licensed under either of:

- Apache License, Version 2.0 ([LICENSE-APACHE](LICENSE-APACHE))
- MIT license ([LICENSE-MIT](LICENSE-MIT))

at your option.
