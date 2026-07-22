# p (Print Command) and -n Option in sed

## Overview

The `p` command and the `-n` option are the two building blocks of using `sed` as a **line selector** (a `grep`-like filter). On their own each has a distinct role; combined as `sed -n '...p'` they let you print exactly the lines you want — by pattern, line number, range, or interval — and nothing else.

> [!TIP]
> The idiom to remember is `sed -n '<selector>p'`: `-n` turns off automatic printing, and `p` prints only what the selector matches. Drop the `-n` and every matched line prints **twice** (once automatically, once from `p`).

---

## What does `p` do?

- `p` tells `sed` to print the current pattern space (current line).


---

## What does `-n` do?

- `-n` suppresses automatic printing of lines.

- Without `-n`, `sed` prints every line automatically.

- With `-n`, `sed` prints only lines explicitly requested using `p`.


---

## Example File: `user-list.txt`

```bash
vim user-list.txt
```

```txt
User_Name	UID		Dep		Shell
root		0		admin		/bin/bash
armour		1000	admin		/bin/bash
		armour
user1		1001	emp		/bin/bash
user2		1002	emp		/bin/bash
rahul		1003	hr		/bin/bash
```

---

## Basic Print Examples

- Print Lines Matching `root` Without `-n`

```bash
sed '/root/p' user-list.txt
```

> Explanation:
> `sed` automatically prints all lines
> `p` prints matched line again
> Matching lines appear twice
> Output:

```txt
User_Name	UID		Dep		Shell
root		0		admin		/bin/bash
root		0		admin		/bin/bash
armour		1000	admin		/bin/bash
		armour
user1		1001	emp		/bin/bash
user2		1002	emp		/bin/bash
rahul		1003	hr		/bin/bash
```

- Print Lines Matching `root` With `-n`

```bash
sed -n '/root/p' user-list.txt
```

> Output:

```txt
root		0		admin		/bin/bash
```

---

## Print Specific Line

- Print Line 3

```bash
sed -n '3p' user-list.txt
```

> Output:

```txt
armour		1000	admin		/bin/bash
```

---

## Print Range of Lines

- Print Lines 2 to 5

```bash
sed -n '2,5p' user-list.txt
```

> Output:

```txt
root		0		admin		/bin/bash
armour		1000	admin		/bin/bash
		armour
user1		1001	emp		/bin/bash
```

---

## Negation with `!`

- Print All Lines Except 1 to 3

```bash
sed -n '1,3!p' user-list.txt
```

> Output:

```txt
		armour
user1		1001	emp		/bin/bash
user2		1002	emp		/bin/bash
rahul		1003	hr		/bin/bash
```

---

## Print From Line Number to End

- Print From Line 2 to End

```bash
sed -n '2,$p' user-list.txt
```

> Output:

```txt
root		0		admin		/bin/bash
armour		1000	admin		/bin/bash
		armour
user1		1001	emp		/bin/bash
user2		1002	emp		/bin/bash
rahul		1003	hr		/bin/bash
```

- Print Everything Except From Line 2 to End

```bash
sed -n '2,$!p' user-list.txt
```

> Output:

```txt
User_Name	UID		Dep		Shell
```

---

## Print Multiple Specific Lines

- Print Line 3 and Line 5

```bash
sed -n -e '3p' -e '5p' user-list.txt
```

> Output:

```txt
armour		1000	admin		/bin/bash
user1		1001	emp		/bin/bash
```

Explanation:

|Option|Meaning|
|---|---|
|`-n`|Disable automatic printing|
|`-e '3p'`|Print line 3|
|`-e '5p'`|Print line 5|

---

## Multiple Commands Using `;`

- Print Lines Matching `root` or `hr`

```bash
sed -n '/root/p;/hr/p' user-list.txt
```

> Output:

```txt
root		0		admin		/bin/bash
rahul		1003	hr		/bin/bash
```

---

## Using Extended Regular Expressions

- GNU sed

```bash
sed -rn '/root|hr/p' user-list.txt
```

- BSD/macOS sed

```bash
sed -En '/root|hr/p' user-list.txt
```

> Output:

```txt
root		0		admin		/bin/bash
rahul		1003	hr		/bin/bash
```

---

## Print Empty Lines

- Print Blank Lines Only

```bash
sed -n '/^$/p' user-list.txt
```

- Print Non-Empty Lines

```bash
sed -n '/./p' user-list.txt
```

---

## Print Lines Starting with Specific Text

- Print Lines Starting with `user`

```bash
sed -n '/^user/p' user-list.txt
```

> Output:

```txt
user1		1001	emp		/bin/bash
user2		1002	emp		/bin/bash
```

- Print Lines Ending with `/bin/bash`

```bash
sed -n '/\/bin\/bash$/p' user-list.txt
```

---

## Print Using Line Intervals

- Print Every Second Line

```bash
sed -n '2~2p' user-list.txt
```

---

## Combine Printing with Substitution

- Print Only Modified Lines

```bash
sed -n 's/admin/ADMIN/p' user-list.txt
```

> Output:

```txt
root		0		ADMIN		/bin/bash
armour		1000	ADMIN		/bin/bash
```

---

## Print Line Numbers

- Print Line Numbers Only

```bash
sed -n '=' user-list.txt
```

- Print Line Numbers with Content

```bash
sed = user-list.txt | sed 'N;s/\n/\t/'
```

> Example Output:

```txt
1	User_Name	UID		Dep		Shell
2	root		0		admin		/bin/bash
3	armour		1000	admin		/bin/bash
```

---

## Quick Reference Table

|Command|Description|
|---|---|
|`sed -n '5p' file`|Print line 5|
|`sed -n '1,5p' file`|Print lines 1 to 5|
|`sed -n '/root/p' file`|Print matching lines|
|`sed -n '/root/!p' file`|Print non-matching lines|
|`sed -n '$p' file`|Print last line|
|`sed -n '2,$p' file`|Print from line 2 to end|
|`sed -n '/^$/p' file`|Print blank lines|
|`sed -n '/./p' file`|Print non-empty lines|
|`sed -n '2~2p' file`|Print every second line|
|`sed -n '=' file`|Print line numbers|

## Related
- [sed](sed.md) — parent sed command
- [d-(delete-command)](d-(delete-command).md) — inverse select/delete command
- [q-(quit-command)](q-(quit-command).md) — stop processing after a match
- [grep-Command](grep-Command.md) — alternative line-selection tool
- [String-Processing](String-Processing.md) — text-processing hub
- [Linux Administration & Server Hardening](../Readme.md) — course hub
