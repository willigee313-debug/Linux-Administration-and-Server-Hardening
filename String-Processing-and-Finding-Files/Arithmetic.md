# Arithmetic

## Overview

AWK is not just a text extractor — it is a full programming language with built-in numeric support. It can perform arithmetic directly on fields (`$1`, `$3`, …), on variables, and on literals, with no explicit type declarations. Because fields are automatically treated as numbers in a numeric context, AWK is the fastest way to add columns, compute averages, or transform UID values while parsing a file such as `/etc/passwd`.

> [!NOTE]
> A field that "looks" numeric (e.g. `1000`) is used as a number in arithmetic and as a string when concatenated. AWK converts on demand, so `$3 + 1` and `$1 " " $3` both work on the same record.

## Concepts

Common arithmetic operators:

| Operator | Description |
|---|---|
| `+` | Addition |
| `-` | Subtraction |
| `*` | Multiplication |
| `/` | Division |
| `%` | Modulus (Remainder) |
| `^` | Exponentiation (Power) |
| `++` | Increment |
| `--` | Decrement |

## Commands

### Basic arithmetic

- Addition

```bash
echo 4 6 | awk '{ print $1 + $2 }'
```

- Subtraction

```bash
echo 4 6 | awk '{ print $1 - $2 }'
```

- Multiplication

```bash
echo 4 6 | awk '{ print $1 * $2 }'
```

- Division

```bash
echo 100 2 | awk '{ print $1 / $2 }'
```

### Modulus (Remainder)

- Calculates the remainder of `$1` divided by `$2`.

```bash
echo 10 3 | awk '{ print $1 % $2 }'
```

### Exponentiation (Power)

- Calculates `$1` raised to the power of `$2`.

```bash
echo 2 5 | awk '{ print $1 ^ $2 }'
```

### Increment and Decrement

- Post-increment the value of `$1`.

```bash
echo 5 | awk '{ $1++; print $1 }'
```

- Post-decrement the value of `$1`.

```bash
echo 5 | awk '{ $1--; print $1 }'
```

- Increment inside print.

```bash
echo 5 | awk '{ print $1 + 1 }'
```

> [!TIP]
> `$1++` modifies the field in place (and rebuilds `$0`), whereas `print $1 + 1` computes a value without changing the field. Choose based on whether you need the change to persist in the record.

## Examples

### Conditional arithmetic (ternary operator)

The ternary operator gives you a compact `if/else` inside an expression.

> Syntax:

```awk
condition ? value_if_true : value_if_false
```

- Print the larger number.

```bash
echo 10 20 | awk '{ print ($1 > $2) ? $1 : $2 }'
```

- Check even or odd.

```bash
echo 5 | awk '{ print ($1 % 2 == 0) ? "Even" : "Odd" }'
```

### Variables in AWK

- Store sum in a variable.

```bash
echo 10 20 | awk '{ x = $1 + $2; print x }'
```

- Store product in a variable.

```bash
echo 4 5 | awk '{ mul = $1 * $2; print mul }'
```

### Arithmetic with BEGIN block

Perform arithmetic without input. The `BEGIN` block runs once before any line is read, so it needs no input stream.

- Addition example.

```bash
awk 'BEGIN { print 10 + 20 }'
```

- Multiplication example.

```bash
awk 'BEGIN { print 5 * 5 }'
```

### Using multiple variables

```bash
echo 10 5 | awk '{
    sum = $1 + $2
    sub = $1 - $2
    mul = $1 * $2
    div = $1 / $2

    print "Sum:", sum
    print "Subtraction:", sub
    print "Multiplication:", mul
    print "Division:", div
}'
```

### Average calculation example

- Calculate average of two numbers.

```bash
echo 10 20 | awk '{ avg = ($1 + $2) / 2; print avg }'
```

### Square and cube example

- Square

```bash
echo 5 | awk '{ print $1 ^ 2 }'
```

- Cube

```bash
echo 5 | awk '{ print $1 ^ 3 }'
```

### Practical example with /etc/passwd

- Print username and UID increased by 1.

```bash
awk -F: '{ print $1, $3 + 1 }' /etc/passwd
```

- Filter users with UID greater than or equal to 1000 and print doubled UID value.

```bash
awk -F: '$3 >= 1000 { print $1, $3 * 2 }' /etc/passwd
```

> [!NOTE]
> UIDs `>= 1000` typically identify regular user accounts, so this pattern is a quick way to run calculations across only the human users during a host audit.

## Best Practices

- Force numeric context with `+0` (e.g. `$3+0 > 1000`) when a field might contain surrounding whitespace or non-numeric junk.
- Use named variables for multi-step calculations — they read better than deeply nested expressions.
- Guard divisions against a zero denominator to avoid `division by zero` errors.
- Keep heavy numeric logic in a `-f script.awk` file rather than a cramped one-liner.

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| String is concatenated instead of added | Field used in string context | Force numeric: `$1+0 + $2+0` |
| `division by zero` error | Denominator field is `0`/empty | Test the value before dividing |
| Unexpected large/decimal result | Integer vs floating-point | AWK uses floating point; format with `printf "%d"` if integers wanted |
| Comparison never true | Numeric field compared as string | Add `+0` to force numeric comparison |

## References

- GNU AWK User's Guide — Arithmetic Operators: <https://www.gnu.org/software/gawk/manual/html_node/Arithmetic-Ops.html>

## Related
- [awk-Command](awk-Command.md) — awk language overview
- [$1-$2-Dollars-everywhere]($1-$2-Dollars-everywhere.md) — fields used in calculations
- [print-BEGIN{}-{}-END{}](print-BEGIN{}-{}-END{}.md) — output computed results
- [Number-of-Fields](Number-of-Fields.md) — field counts in expressions
- [String-Processing](String-Processing.md) — text-processing hub
- [Linux Administration & Server Hardening](../Readme.md) — Linux administration & server hardening hub
