# baker

Small Bash scripts for quickly making timestamped archives and comparing files and directories
against them.

| Script | Purpose                                                            |
| ------ | ------------------------------------------------------------------ |
| `qfb`  | Compress single files (gzip, xz, or zstd) into timestamped copies. |
| `qtar` | Bundle files and directories into a timestamped tar archive.       |
| `qzip` | Bundle files and directories into a timestamped zip archive.       |

Archives are written to `/var/tmp` by default and named like
`quickfile-NAME-YYYY-MM-DD_HH-MM.gz` or `quicktar-NAME-YYYY-MM-DD_HH-MM.tar.zst`.

## Requirements

- Bash
- The tools for the formats you use: `gzip`, `xz`, `zstd`, `bzip2`, `tar`, `zip`/`zipinfo`,
  `sha256sum`
- `diff` for `qfb --diff` and `qtar --diff`, plus the decompressor for the archive format being
  read.

## Usage

Run `-h` on any script for its full option list.

```bash
# Compress a single file (keeps the original)
./qfb notes.txt

# Compress with xz, test the result, and remove the original
./qfb -c xz -t -r notes.txt

# Archive a directory as tar.zst with a checksum
./qtar -s projects/

# Archive a directory and file as a zip, excluding *.log
./qzip -x '*.log' projects/ README.md

# Show what would be archived without writing anything
./qtar -N projects/

# Compare a file with its latest qfb archive
./qfb --diff notes.txt

# Compare a directory with its latest qtar archive
./qtar --diff projects/
```

## Notes

- Timestamps have minute resolution, so two runs in the same minute overwrite each other's output.
- `qfb --diff` compares files and `qtar --diff` compares directories, in both cases against the
  newest archive matching the name, searched in the `-o` directory (`/var/tmp` by default).
  Neither reads `qzip` archives.
- Inputs over 500 MiB switch `qtar` and `qzip` to verbose mode automatically.
