# Paste Command in Linux

## Overview

- The `paste` command is used to merge lines from multiple files horizontally, combining corresponding lines side by side.

- It is commonly used for:

	- Combining columns from multiple files

	- Creating CSV-style output

	- Joining related datasets

	- Formatting text output


---

## Syntax

```bash
paste [OPTION] [FILE]
```

> If no file is specified, `paste` reads from standard input.

---

## Sample Input Files

- Create First Name File

```bash
vim fname.txt
```

```text
Ankit
Ayush
Dhreeraj
Gourav
```

- Create Last Name File

```bash
vim lname.txt
```

```text
Sharma
Jain
Chauhan
Joshi
```

---

## Basic Usage

### Merge Files Using Default Delimiter (TAB)

- By default, `paste` uses a TAB character between merged columns.

```bash
paste fname.txt lname.txt
```

> Example Output

```text
Ankit      Sharma
Ayush      Jain
Dhreeraj   Chauhan
Gourav     Joshi
```

### Merge Using Space Delimiter

```bash
paste -d " " fname.txt lname.txt
```

> Output

```text
Ankit Sharma
Ayush Jain
Dhreeraj Chauhan
Gourav Joshi
```

### Merge Using Hyphen

```bash
paste -d "-" fname.txt lname.txt
```

> Output

```text
Ankit-Sharma
Ayush-Jain
Dhreeraj-Chauhan
Gourav-Joshi
```

### Merge Using Underscore

```bash
paste -d "_" fname.txt lname.txt
```

> Output

```text
Ankit_Sharma
Ayush_Jain
Dhreeraj_Chauhan
Gourav_Joshi
```

### Merge Using Slash

```bash
paste -d "/" fname.txt lname.txt
```

> Output

```text
Ankit/Sharma
Ayush/Jain
Dhreeraj/Chauhan
Gourav/Joshi
```

---

### Empty Delimiter Usage

```bash
paste -d "" fname.txt lname.txt
```

### Save Output to a File

- Merge using a space delimiter and save the result.

```bash
paste -d " " fname.txt lname.txt > fullname.txt
```

- View Saved File

```bash
cat fullname.txt
```

---

## Working with Multiple Files

- Create Third File

```bash
vim age.txt
```

```text
25
26
24
27
```

- Merge Three Files

```bash
paste -d "," fname.txt lname.txt age.txt
```

> Output

```text
Ankit,Sharma,25
Ayush,Jain,26
Dhreeraj,Chauhan,24
Gourav,Joshi,27
```

## Serial Mode (`-s`)

- The `-s` option merges one file at a time instead of line by line.

```bash
paste -s fname.txt
```

> Output

```text
Ankit    Ayush    Dhreeraj    Gourav
```

- Serial Mode with Custom Delimiter

```bash
paste -s -d "," fname.txt
```

> Output

```text
Ankit,Ayush,Dhreeraj,Gourav
```

---

## Using `paste` with Pipes

- Merge Command Output

```bash
echo -e "A\nB\nC" | paste -s -d ","
```

> Output

```text
A,B,C
```

## Combine Username and Shell from `/etc/passwd`

```bash
cut -d ":" -f1 /etc/passwd > users.txt
cut -d ":" -f7 /etc/passwd > shells.txt
paste -d ":" users.txt shells.txt
```

-  Create CSV Data

```bash
paste -d "," fname.txt lname.txt > users.csv
```

---

## Useful Options

|Option|Description|
|---|---|
|`-d`|Specify delimiter|
|`-s`|Serial mode|
|`-`|Read from standard input|

---

## Notes

- `paste` combines files horizontally.

- By default, columns are separated using TAB characters.

- Multiple delimiters can be specified:


```bash
paste -d ",:" file1 file2 file3
```

- `paste` is commonly used with:

    - `cut`

    - `awk`

    - `sort`

    - `uniq`

    - `grep`

- For more advanced merging and formatting, `awk` may provide greater flexibility.

---

## Best Practices

- Ensure input files have the **same number of lines**; `paste` aligns by line position, so mismatched line counts produce blank fields on the shorter side.

- Choose a delimiter that will not appear inside the data itself (e.g. avoid `-d ","` when field values already contain commas) to keep generated CSV parseable.

- When building datasets programmatically, write to a file with `>` and validate the result with `cat` or `head` before consuming it downstream.

- Prefer `awk` when you need conditional logic, reordering, or per-field transformation rather than a straight positional merge.

> [!TIP]
> Use `paste -s` to collapse a multi-line list into a single delimited line — handy for turning command output into a comma-separated string.

---

## Security Considerations

- Merging fields from `/etc/passwd` (as in the username/shell example) exposes account metadata. Treat any generated files as potentially sensitive and store them with restrictive permissions (`chmod 600`).

- Do not paste secrets (tokens, keys, hashes) into world-readable output files or shared directories; the merged result is only as protected as the file you write it to.

- When generating CSV/report data from system files, sanitise or review the output before distribution to avoid leaking account names, shells, or UIDs beyond their intended audience.

---

## Related
- [String-Processing](String-Processing.md) — parent text-processing hub
- [cut-Command](cut-Command.md) — inverse: split columns that paste joins
- [awk-Command](awk-Command.md) — column-aware text processing
- [Field-Separator](Field-Separator.md) — controls delimiters like paste -d
- [Linux Administration & Server Hardening](../Readme.md) — Linux commands hub
