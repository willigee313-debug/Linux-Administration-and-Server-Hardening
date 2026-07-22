# Number of Fields (`NF`)

## Overview

`NF` is a built-in `awk` variable that holds the **Number of Fields** in the current input record (line). Because `awk` splits every record into fields automatically, `NF` lets you reference fields relative to the end of the line — most usefully `$NF` for the **last field** and `$(NF-1)` for the **second-to-last** — without knowing the column count in advance.

By default, fields are separated by runs of whitespace (spaces or tabs). A custom delimiter is set with the `-F` option (for example `-F ":"` for `/etc/passwd`).

> [!TIP]
> **Value vs. reference**
> `NF` (no `$`) is the **count** of fields. `$NF` (with `$`) is the **value** of the field at that position — i.e. the last field. `$(NF-1)`, `$(NF-2)`, … walk backward from the end.

## Concepts

| Expression | Meaning |
|---|---|
| `NF` | Number of fields in the current record |
| `$NF` | Value of the last field |
| `$(NF-1)` | Second-to-last field |
| `$(NF-2)` | Third-to-last field |
| `$1`, `$4` | First / fourth field (counted from the start) |
| `$(NF+1)` | A field past the end — does not exist, prints blank |

> [!NOTE]
> **Out-of-range fields are blank, not errors**
> Referencing a field that does not exist (e.g. `$(NF+1)` or `$(NF-4)` on a 4-field line) yields an **empty value** rather than an error.

## Basic `NF` Examples

- Prints the **number of fields** in the input line.

```bash
echo "one two three four" | awk '{print NF}'
```

- Prints the **last field**.

```bash
echo "one two three four" | awk '{print $NF}'
```

- Prints the **fourth field**.

```bash
echo "one two three four" | awk '{print $4}'
```

- Prints the **4th and 1st fields**.

```bash
echo "one two three four" | awk '{print $4,$1}'
```

- Prints the **second-to-last field**.

```bash
echo "one two three four" | awk '{print $(NF-1)}'
```

- Prints the **third-to-last field**.

```bash
echo "one two three four" | awk '{print $(NF-2)}'
```

- Prints the **fourth-to-last field**.

```bash
echo "one two three four" | awk '{print $(NF-3)}'
```

- Prints the **fifth-to-last field**, which does not exist (blank output).

```bash
echo "one two three four" | awk '{print $(NF-4)}'
```

- Prints the **field after the last** (non-existent, blank output).

```bash
echo "one two three four" | awk '{print $(NF+1)}'
```

## AWK with `/etc/passwd`

The `/etc/passwd` file uses `:` as the field separator, so pass `-F ":"` to count and address its columns correctly.

- Prints the **number of fields** in each line.

```bash
cat /etc/passwd | awk '{print NF}'
```

- Prints the **number of fields** using `:` as delimiter.

```bash
cat /etc/passwd | awk -F ":" '{print NF}'
```

- Preferred syntax without `cat`.

```bash
awk -F":" '{print NF}' /etc/passwd
```

- Prints the **last field** using `:` as delimiter.

```bash
cat /etc/passwd | awk -F ":" '{print $NF}'
```

- Preferred syntax without `cat`.

```bash
awk -F":" '{print $NF}' /etc/passwd
```

- Prints the **first field**, **second-to-last field**, and **last field**.

```bash
cat /etc/passwd | awk -F ":" '{print $1,$(NF-1),$NF}'
```

- Prints the **second-to-last field**.

```bash
awk -F":" '{print $(NF-1)}' /etc/passwd
```

- Alternative syntax.

```bash
cat /etc/passwd | awk -F ":" '{print $(NF-1)}'
```

- Displays text with the **number of fields** in each line.

```bash
cat /etc/passwd | awk '{print "Number of fields in this line: " NF}'
```

- With `:` delimiter.

```bash
cat /etc/passwd | awk -F ":" '{print "Number of fields in this line: " NF}'
```

> [!TIP]
> **Avoid useless `cat`**
> Piping `cat file | awk ...` works, but `awk` reads files directly. Prefer `awk '...' file` — it is one less process and the idiomatic form.

## AWK with `ps -aux`

- Prints the **second-to-last field** from the process list.

```bash
ps -aux | awk '{print $(NF-1)}'
```

- Prints the **last field** from the process list.

```bash
ps -aux | awk '{print $NF}'
```

- Prints the **number of fields** in each process line.

```bash
ps -aux | awk '{print NF}'
```

## AWK with `ifconfig`

- Prints the **number of fields** in each line.

```bash
ifconfig | awk '{print NF}'
```

- Prints the **last field**.

```bash
ifconfig | awk '{print $NF}'
```

- Prints the **second-to-last field**.

```bash
ifconfig | awk '{print $(NF-1)}'
```

- Prints the **last and second-to-last fields**.

```bash
ifconfig | awk '{print $(NF-1),$NF}'
```

- Alternate syntax (same result).

```bash
ifconfig | awk '{print $(NF-1),$(NF)}'
```

## Best Practices

- `NF` is automatically updated for every input line — you never set it manually to count fields.
- `$NF` always means the **last field**; `$(NF-1)` the **second-to-last**. Use these instead of hard-coding column numbers when the last columns are what you want.
- If the referenced field does not exist, `awk` prints an empty value — guard against blank output when field counts vary.
- Use `-F` to define a custom delimiter that matches your data (`:` for `/etc/passwd`, `,` for CSV).
- Prefer `awk '...' file` over `cat file | awk '...'`.

> Example:

```bash
awk -F ":" '{print $1}' /etc/passwd
```

## Related
- [awk-Command](awk-Command.md) — awk language overview
- [Number-of-Records](Number-of-Records.md) — companion `NR` variable
- [Field-Separator](Field-Separator.md) — controls how fields are counted
- [$1-$2-Dollars-everywhere]($1-$2-Dollars-everywhere.md) — accessing individual fields
- [Linux Administration & Server Hardening](../Readme.md) — course hub
