# baker

Small Bash scripts for quickly making timestamped archives and comparing files against them.

| Script | Purpose |
| --- | --- |
| `qfb` | Compress individual files (gzip, xz, or zstd) into timestamped copies. |
| `qtar` | Bundle files and directories into a timestamped tar archive. |
| `qzip` | Bundle files and directories into a timestamped zip archive. |
| `qdiff` | Diff a file or directory against its most recent `qfb` or `qtar` archive. |

Archives are written to `/var/tmp` by default and named like `quickfile-NAME-YYYY-MM-DD_HH-MM.gz` or `quicktar-NAME-YYYY-MM-DD_HH-MM.tar.zst`.

## Requirements

- Bash
- The tools for the formats you use: `gzip`, `xz`, `zstd`, `bzip2`, `tar`, `zip`/`zipinfo`, `sha256sum`
- `qdiff` needs the same decompressors as the archives it reads.

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
./qdiff notes.txt

# List only the changed file names in a directory
./qdiff -s projects/
```

## Notes

- Timestamps have minute resolution, so two runs in the same minute overwrite each other's output.
- `qdiff` uses the newest archive matching the name it is given, and looks in `/var/tmp` unless `-d DIR` is set. It does not read `qzip` archives.
- Inputs over 500 MiB switch `qtar` and `qzip` to verbose mode automatically.
