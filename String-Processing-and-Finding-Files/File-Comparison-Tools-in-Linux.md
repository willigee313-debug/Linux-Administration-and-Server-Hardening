# File Comparison Tools in Linux

## Overview

Comparing files is a routine task in Linux administration, security auditing, and change management. Whether you are validating that a configuration file matches a known-good baseline, generating a patch, or reviewing what changed between two versions of a script, Linux ships several purpose-built comparison tools.

This note covers the three most common tools — `comm`, `diff`, and `vimdiff` — along with when to reach for each. In short: use `comm` for **sorted set operations** (what is unique to each file, what they share), `diff` for **line-by-line change reporting and patch generation**, and `vimdiff` for **interactive side-by-side review and merging**.

> [!TIP]
> **Choosing a tool**
> - **Set membership** (which lines are unique or shared) → `comm`
> - **Automated diffs, patches, scripting** → `diff`
> - **Manual review and merging** → `vimdiff`
> - **Binary / byte-exact equality** → `cmp`

## Concepts

| Tool | Comparison model | Input requirement | Best for |
|---|---|---|---|
| `comm` | Set-based, three columns | Both files must be **sorted** | Finding lines unique to each file or common to both |
| `diff` | Line-by-line change hunks | None (text files) | Patches, config drift, source review, scripting |
| `vimdiff` | Visual, in-editor | None | Interactive review and merge editing |

## comm

The `comm` command compares two **sorted** files line-by-line and reports the result in three columns:

1. Lines only in the first file
2. Lines only in the second file
3. Lines common to both files

Numeric flags **suppress** the matching column, letting you isolate exactly the set you care about.

### Basic Syntax

```bash
comm [options] file1 file2
```

### Important Requirement

Both input files must be sorted before using `comm`. Unsorted input produces misleading output because `comm` walks both files in lockstep and assumes sorted order.

> [!WARNING]
> **Sort first**
> `comm` does **not** sort for you. If the files are not already sorted, sort them (or pipe pre-sorted input) or the column assignments will be wrong.

```bash
sort 1.txt -o 1.txt
```

```bash
sort 2.txt -o 2.txt
```

### Common Examples

- Compare two sorted files

```bash
comm 1.txt 2.txt
```

- Suppress column 1 (lines unique to first file)

```bash
comm -1 1.txt 2.txt
```

- Suppress column 2 (lines unique to second file)

```bash
comm -2 1.txt 2.txt
```

- Suppress column 3 (common lines)

```bash
comm -3 1.txt 2.txt
```

- Show only common lines

```bash
comm -12 1.txt 2.txt
```

- Show only lines unique to first file

```bash
comm -23 1.txt 2.txt
```

- Show only lines unique to second file

```bash
comm -13 1.txt 2.txt
```

> [!NOTE]
> **Reading the flags**
> Each digit names the column to **hide**. So `-12` hides columns 1 and 2, leaving only column 3 (common lines); `-23` leaves only column 1 (unique to the first file).

## diff

The `diff` command compares files line-by-line and displays the differences between them. It is the workhorse for source-code review, configuration comparisons, and patch generation.

### Basic Syntax

```bash
diff [options] file1 file2
```

### Common Options

| Option | Purpose |
|---|---|
| `-c` | Context diff — changes with surrounding context lines |
| `-u` | Unified diff — the format used by patches and Git |
| `-y` | Side-by-side comparison |
| `-w` | Ignore whitespace differences |
| `-i` | Ignore case differences |
| `-r` | Compare directories recursively |
| `-q` | Report only whether files differ (quiet) |

### Common Examples

- Standard file comparison

```bash
diff 1.txt 2.txt
```

- Context diff: Shows changes with surrounding context lines.

```bash
diff -c 1.txt 2.txt
```

- Commonly used in patches and Git.

```bash
diff -u 1.txt 2.txt
```

- Side-by-side comparison

```bash
diff -y file1 file2
```

- Ignore whitespace differences

```bash
diff -w file1 file2
```

- Ignore case differences

```bash
diff -i file1 file2
```

- Compare directories recursively

```bash
diff -r dir1 dir2
```

- Show only whether files differ

```bash
diff -q file1 file2
```

### Understanding `diff` Output

The default (normal) `diff` format uses change commands like `3c3` (change), `a` (add), and `d` (delete):

```text
3c3
< old line
---
> new line
```

Meaning:

- Line 3 changed
- `<` indicates content from the first file
- `>` indicates content from the second file

## vimdiff

The `vimdiff` command opens files side-by-side inside Vim with differences highlighted. It is ideal for interactive comparison and editing, letting you pull changes between windows.

### Basic Syntax

```bash
vimdiff file1 file2
```

### Common Examples

- Compare two files

```bash
vimdiff 1.txt 2.txt
```

- Compare three files

```bash
vimdiff file1 file2 file3
```

### Useful Vimdiff Commands

|Command|Description|
|---|---|
|`]c`|Jump to next difference|
|`[c`|Jump to previous difference|
|`:diffget`|Get changes from another window|
|`:diffput`|Send changes to another window|
|`:qa`|Quit all windows|
|`:wqa`|Save and quit all|

## Examples

Real-world tasks that combine these tools with everyday administration.

- Compare configuration files

```bash
diff /etc/ssh/sshd_config backup_sshd_config
```

- Compare two directory trees

```bash
diff -r dir_old dir_new
```

- Compare sorted user lists

```bash
comm users1.txt users2.txt
```

- Visual comparison of source code

```bash
vimdiff app_old.py app_new.py
```

### Common Command Combinations

- `sort` + `comm`

```bash
sort file1.txt -o file1.txt && sort file2.txt -o file2.txt && comm file1.txt file2.txt
```

- `diff` + `grep`

```bash
diff file1 file2 | grep "^>"
```

- `diff` + `less`

```bash
diff -u old.conf new.conf | less
```

- `vimdiff` in read-only mode

```bash
vimdiff -R file1 file2
```

## Related Commands

|Command|Description|
|---|---|
|`cmp`|Byte-by-byte file comparison|
|`sdiff`|Side-by-side merge tool|
|`colordiff`|Colored wrapper for `diff`|
|`git diff`|Git-integrated file comparison|

## Best Practices

- `comm` is very fast but requires **sorted** input — sort both files first or its output is meaningless.
- Use `diff` for text comparisons and prefer the unified format (`-u`) so the output feeds cleanly into `patch` and version control.
- `vimdiff` is interactive and better for manual review and editing when you need to selectively merge changes.
- Large or binary files are better compared with `cmp`, which stops at the first differing byte.
- When validating configuration against a baseline, pair `diff -q` (fast "differ or not") in scripts with a full `diff -u` on the hits for detail.

## Security Considerations

- Avoid comparing sensitive files (keys, credentials, shadow entries) in shared or logged terminals — the contents scroll into scrollback and session logs.
- `vimdiff` may create swap files (`.swp`) that persist copies of the content; disable them (`-i NONE`, or `set noswapfile`) when inspecting secrets.
- Use read-only mode (`vimdiff -R`) when inspecting critical configuration files so you cannot accidentally save changes.
- Baseline comparisons against a known-good copy are a core file-integrity control — align them with CIS Benchmark configuration checks and file-integrity monitoring (FIM) practices.

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| `comm` output looks scrambled | Input files not sorted | `sort file -o file` both inputs first |
| `diff` reports differences you cannot see | Whitespace or line-ending differences | Use `diff -w` or normalize line endings (`dos2unix`) |
| `vimdiff` leaves `.swp` files behind | Editor swap files enabled | Quit cleanly with `:qa`, or launch with `-i NONE` |
| Directory compare too noisy | Comparing generated/binary files | Add `-x` exclude patterns or restrict to text with `-r --brief` |

## Help

- `comm`

```bash
man comm
```

- `diff`

```bash
man diff
```

- `vimdiff`

```bash
man vimdiff
```

## Related
- [String-Processing](String-Processing.md) — text-processing toolkit overview
- [grep-Command](grep-Command.md) — filter `diff` output or search inside files
- [File-Finding-in-Linux](File-Finding-in-Linux.md) — locate the files you want to compare
- [Linux Administration & Server Hardening](../Readme.md) — course hub
