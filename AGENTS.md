# AGENTS.md

Bash-only repo: four standalone scripts at the root (`qfb`, `qtar`, `qzip`, `qdiff`) and `README.md`, which has the usage examples. There is no build, package manager, test suite, linter, or CI config.

## Verifying changes

- Syntax check: `bash -n qfb qtar qzip qdiff`. `shellcheck` is not installed here.
- Use `-d` to point output at a scratch directory such as `/tmp/opencode/...`. The default output dir is `/var/tmp`, and `qdiff` searches there, so test archives left in `/var/tmp` can affect later `qdiff` results for the same name.
- `-N` (dry run) is available on `qfb`, `qtar`, and `qzip`. `qdiff` is read-only.
- Round-trip check: `qfb -d DIR f && qdiff -d DIR f` should print `f matches ...` and exit 0. After a change, `qdiff` exits 1 and prints a unified diff.
- Do not use `qfb -r` on real files while testing. It deletes the source.

## Architecture

- Each script is self-contained. `usage`, `die`, option parsing, color detection, and the `human_size` awk block are copy-pasted across scripts. A change to shared behavior (size format, option letters, exit codes) must be applied to every script that has it.
- Archive names are a contract between the writers and `qdiff`. Writers use the sanitizer `[^A-Za-z0-9._-]` -> `_` and a `YYYY-MM-DD_HH-MM` stamp:
  - `qfb`: `quickfile-NAME-STAMP.{gz,xz,zst}`
  - `qtar`: `quicktar-NAME-STAMP.tar.{zst,gz,xz,bz2}` or `.tar`
  - `qzip`: `quickzip-NAME-STAMP.zip`
- `qdiff` matches a file against `quickfile-*` archives only, and a directory against `quicktar-*` archives only. It never reads `qzip` archives. If you change a prefix, extension, or stamp format, update `qdiff` too.
- Stamps have minute resolution. Two runs in the same minute overwrite each other's output.

## Behavior gotchas

- `qfb` accepts only regular files. Use `qtar` or `qzip` for directories.
- `-s` (SHA-256 sidecar) exists on `qtar` and `qzip`, not `qfb`. `qdiff -s` only affects directory output and is silently ignored for files.
- `qfb` takes `-c gz|xz|zst`. `qtar` takes `-c zst|gz|xz|bz2|none`; `-l` is ignored with `none`. `qzip` has no `-c` or `-l`.
- Exclude patterns are passed through: `qtar -x` becomes tar `--exclude=`, and `qzip -x` becomes zip's `-x` wildcard list, which must come last. Quote patterns in examples.
- `qtar` and `qzip` switch to verbose output automatically for inputs of 500 MiB or more.
- Dependencies are checked at runtime with `command -v`: `gzip`, `xz`, `zstd`, `bzip2`, `tar`, `zip`/`zipinfo`, and `sha256sum`. `qdiff` needs the decompressor for each archive format it reads.
- Scripts use `set -euo pipefail`. On failure, `qfb`, `qtar`, and `qzip` delete partial output before exiting.

## Repo state

- This is a git repo with a short history (`First commit`, `Add a README`, `Add a version string`). Commit messages are short imperative sentences.
