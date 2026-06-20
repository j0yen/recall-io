# recall-io

Backup, restore, and migration for a [recall](https://github.com/j0yen/recall) memory store: serialize the whole store to NDJSON and load it back, round-trip-faithful.

A recall store is a tree of Markdown files plus a SQLite index — convenient to live in, awkward to move. Copying it between machines means copying a directory and a database and trusting they stay consistent; editing many memories at once means opening many files. `recall-io` collapses the store into one stream: one JSON object per memory, one per line. That single file is easy to back up, diff, hand-edit, or carry to another machine — and `import` reconstructs the files and index from it exactly.

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

Builds and installs the `recall-io` binary with `cargo install --path . --locked` (lands in `~/.cargo/bin/`). Requires `cargo` / `rustc 1.85+` and `git`.

## Usage

```sh
# Back the store up to one file:
recall-io export > recall-backup.jsonl

# Restore (or migrate to another machine):
recall-io import recall-backup.jsonl

# Round-trip from a pipe:
recall-io export | recall-io import -
```

**Export** writes one JSON object per memory to stdout, sorted by id — deterministic, so two exports of the same store are byte-identical and safe to diff. Each record carries the memory's full state: `id`, `kind`, `subject`, `path`, `confidence`, `created_at`, `last_recalled_at`, `recall_count`, `embedding_id`, and the body.

**Import** reads NDJSON (from a file, or `-` for stdin) and writes one `.md` per memory under `<root>/memories/<subject>/<id>.md`, inserting the matching index rows and creating the root and index if they don't exist. It's additive by default: a memory whose id already exists is skipped with a warning, leaving the original untouched. `--replace` overwrites duplicates instead. Malformed or incomplete lines are logged with their line number and counted as errors; import continues past them. The summary — `{imported, skipped, errors}` — prints as one line of text, or as a JSON object with `--format json`.

Both commands default to the `~/.claude/recall` store; override with `--root`.

Exit codes: `0` success; `1` an expected failure (nothing to export, or every import line invalid); `2` an invocation error (bad path, `--root` that isn't a directory).

## Where it fits

Part of the recall family:

- **[recall](https://github.com/j0yen/recall)** — the agentic-memory store this reads and writes.
- **[recall-doctor](https://github.com/j0yen/recall-doctor)** — `fsck` for the same store; run it after an import to confirm the files and index agree.

## Status

Each acceptance criterion has a matching integration test under `tests/acceptance_ac<n>.rs`, including a wipe-and-restore round-trip that checks file count and byte-identical bodies. Built via the [`autobuilder`](https://github.com/j0yen/autobuilder) pipeline (PRD intake → intent-card → scaffold → iterate-and-prove). Originally a subdirectory of the [`wintermute`](https://github.com/j0yen/wintermute) monorepo; this repo is the standalone distribution.

## License

Dual-licensed under MIT OR Apache-2.0; pick whichever fits. See [LICENSE-MIT](LICENSE-MIT) and [LICENSE-APACHE](LICENSE-APACHE).
