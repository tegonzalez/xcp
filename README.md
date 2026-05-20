# xcp

`xcp` is a small `rsync` wrapper for copying large directory trees with explicit, predictable, exclusionary path behavior.

It does two things:

- `xcp ls` helps build an exclude file by showing large retained paths before you copy.
- `xcp cp` delegates to rsync for local and remote (SSH) paths

## Requirements

- Python 3.9 or newer
- `rsync` on `PATH` for `xcp cp`
- SSH or rsync daemon credentials configured for remote copies

## Start

Command `ls` requires a path and a size. The size is the minimum size for consideration. A reasonable value is 1-10% of the size you intend to copy and `ls` identifies large outliers (files and directory trees)

List paths under the current directory that are at least 500 MiB:

```bash
./xcp ls . 500M
```

Example output:

```text
1.4GiB/1.4GiB  14/210  **/node_modules/
820MiB/821MiB   3/48   **/dist/
612MiB/612MiB   1/12   **/.cache/
```

| Column | Meaning |
| --- | --- |
| `logical-size/disk-usage` | File size total and allocated disk usage. |
| `dir-count/file-count` | Number of directories and files represented by the suggested pattern. |
| `suggested-pattern` | Exclude pattern you can copy into `exclude-file`. |

Default sort is by path. Sort biggest paths first:

```bash
./xcp -s size ls . 500M
```

Preview a copy:

```bash
./xcp cp ./project ./backup/ --dry-run
```

Run the copy:

```bash
./xcp cp ./project ./backup/
```

Destination slash matters:

| Command | Result |
| --- | --- |
| `xcp cp project/ backup/` | `backup/project/` |
| `xcp cp project backup/` | `backup/project/` |
| `xcp cp project/ backup` | `backup/` |
| `xcp cp project backup` | `backup/` |

Source slash does not matter. Destination slash chooses whether the destination is a parent directory or the exact copy target.


## Excludes

Create an exclude file named `exclude-file`:

```text
# directories
**/.git/
**/node_modules/
**/__pycache__/

# files
**/*.tmp
**/*.log
```

Use it for both analysis and copy:

```bash
./xcp -x exclude-file ls . 500M
./xcp -x exclude-file cp ./project ./backup/ --dry-run
./xcp -x exclude-file cp ./project ./backup/

./xcp -x exclude-file cp ~/work/project user@server.local:backup/ --dry-run
./xcp -x exclude-file cp ~/work/project user@server.local:backup/
```

Rsync daemon URLs also work:

```bash
./xcp -x exclude-file cp rsync://server.local/module/project ./backup/ --dry-run
```

Remote paths and rsync URLs are passed through to `rsync`, so normal SSH keys, agent auth, password prompts, and rsync daemon URLs still apply.

With excludes, `ls` and `cp` print a summary of total, kept, and excluded size:

```text
total 18GiB/19GiB  kept 11GiB/12GiB  excluded 7.0GiB/7.1GiB
```

Exclude file format:

```text
# comments and blank lines are ignored
**/relative-directory/
**/relative-file-pattern
- **/also-accepted/
```

Rules must be relative. Leading `/`, `+ pattern`, and `!pattern` are rejected.

## Commands

```text
xcp [-x exclude-file] command ...
xcp ls PATH SIZE [-s {path,size}]
xcp cp SOURCE DEST [-n]
```

Global options:

```text
-x, --exclude-from exclude-file  read shared exclude patterns
```

Command options:

```text
xcp ls -s, --sort {path,size}    sort output, default path
xcp cp -n, --dry-run             preview without copying
```
