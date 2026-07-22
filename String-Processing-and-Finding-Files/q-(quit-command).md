# `q` (Quit Command) in `sed`

## Overview

- The `q` command tells `sed` to stop processing input immediately.

- `sed` exits as soon as the specified pattern or line number is matched — remaining lines are never read.

- You can optionally provide an exit code such as `q1`, `q2`, etc.

- The exit status can later be checked using `$?`.

> [!TIP]
> Because `q` halts as soon as it matches, it is an efficient way to "short-circuit" processing of very large files — `sed` stops reading once the condition is met instead of scanning to the end.

---

## Concepts

### How `q` Works

| Form | Behavior |
|---|---|
| `q` | Quit immediately with exit status `0` |
| `qN` | Quit immediately with exit status `N` (e.g. `q1`, `q2`) |
| `/pattern/q` | Quit at the first line matching `pattern` |
| `Nq` | Quit at line number `N` |

- Without `-n`, `sed` prints each line as it is processed, including the matching line, before quitting.

- With `-n`, automatic printing is suppressed, so nothing is printed unless explicitly requested.

> [!NOTE]
> The exit code carried by `qN` becomes the command's exit status, letting shell scripts branch on whether (and where) a pattern was found.

---

## Commands

### Quit on Pattern Match

- Quit processing and exit with code `2` after the first line matching `armour`.

```bash
sed '/armour/q2' user-list.txt
```

### Check Exit Status

- Display the exit status of the previous command.

```bash
echo $?
```

### Suppress Output and Quit

- Suppress automatic printing using `-n` and quit with exit code `2` after matching `armour`.

```bash
sed -n '/armour/q2' user-list.txt
```

### Quit with Different Exit Codes

- Quit with exit code `0` (successful execution) after matching `armour`.

```bash
sed '/armour/q0' user-list.txt
```

- Quit with exit code `1` after matching `armour`.

```bash
sed '/armour/q1' user-list.txt
```

- Quit with exit code `2` after matching `armour`.

```bash
sed '/armour/q2' user-list.txt
```

---

## Best Practices

- Use `q` to stop early when you only need the first match, rather than filtering the entire file — it is faster on large inputs.

- Pair distinct exit codes (`q1`, `q2`, …) with `$?` checks so scripts can distinguish different match conditions.

- Combine `-n` with `q` when you want to control the exit status without emitting any output.

> [!IMPORTANT]
> `q` flushes the current pattern space (prints the line, unless `-n` is set) and then exits. If you need to quit **without** printing the matching line, use `Q` (GNU `sed`) instead of `q`.

---

## Security Considerations

- Exit-code-driven `sed '/pattern/qN'` checks are convenient for validating file contents in hardening scripts (for example, confirming a required directive exists), but they only inspect up to the first match — do not assume the rest of the file is compliant.

- When acting on the result of a `q`-based check, quote and validate any file paths to avoid processing attacker-controlled filenames.

---

## Troubleshooting

| Symptom | Cause | Resolution |
|---|---|---|
| Matching line still printed | `-n` not supplied | Add `-n`, or use `Q` (GNU) to suppress the matched line |
| `$?` is always `0` | No exit code given (plain `q`) | Use `qN` with an explicit non-zero code |
| Whole file processed | Pattern never matched | Verify the regex and the input file contents |

---

## Important Notes

- Without `-n`, `sed` prints lines processed before quitting.

- With `-n`, nothing is printed unless explicitly requested.

- Useful in shell scripting for:

    - early termination,

    - conditional checks,

    - validating file contents,

    - custom exit handling.

## Related
- [sed](sed.md) — parent sed command
- [p-(print-command)-and--n-option](p-(print-command)-and--n-option.md) — often combined to print then quit
- [d-(delete-command)](d-(delete-command).md) — another line-control command
- [String-Processing](String-Processing.md) — text-processing hub
- [Linux Administration & Server Hardening](../Readme.md) — course hub
