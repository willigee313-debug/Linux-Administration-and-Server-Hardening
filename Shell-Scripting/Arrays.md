# Arrays

## Overview

An **array** is a variable that holds an ordered (or keyed) collection of values under a single name. Where a scalar [variable](Variables.md) stores one value, an array stores many — making it the natural structure for lists of hosts, files, ports, or key/value records that a hardening or enumeration script iterates over.

Bash supports two kinds of arrays:

- **Indexed (numeric) arrays** — elements are addressed by integer index starting at `0`.
- **Associative arrays** — elements are addressed by an arbitrary string key (like a dictionary/hash map). Requires Bash 4+.

## Concepts

### Declaration

```bash
ARRAY=(value1 value2 value3)
```

Individual elements are read back by index with the `${ARRAY[index]}` syntax:

```bash
${ARRAY[0]}
${ARRAY[1]}
${ARRAY[2]}
```

### Numeric vs associative arrays

`declare` explicitly types an array before use. Declaring the type is required for associative arrays and recommended for clarity.

| Declaration | Array type |
|-------------|------------|
| `declare -a` | Indexed (numeric) array |
| `declare -A` | Associative array (string keys) |

> [!IMPORTANT]
> Associative arrays (`declare -A`) are only available in **Bash 4.0 and later**. On minimal or legacy systems (older RHEL, embedded/BusyBox shells) this feature may be missing — check with `bash --version` before relying on it in a portable script.

### Syntax reference

| Syntax | Result |
|--------|--------|
| `arr=()` | Create an empty array |
| `arr=(1 2 3)` | Initialize array |
| `arr=(1,2,3)` | Initialize array |
| `${arr[0]}` | Retrieve 1st element |
| `${arr[2]}` | Retrieve 3rd element |
| `${arr[@]}` | Retrieve all elements/items in array |
| `${arr[*]}` | Retrieve all elements/items in array, delimited by first character of IFS |
| `${!arr[@]}` | Retrieve array indices (all indexes in the array, `@`/`*`) |
| `${#arr[@]}` | Calculate array size / number of items in the array (`@`/`*`) |
| `arr[0]=3` | Overwrite 1st element |
| `arr+=(4)` | Append value(s) |
| `str=$(ls)` | Save `ls` output as a string |
| `arr=( $(ls) )` | Save `ls` output as an array of files |
| `${arr[@]:s:n}` | Retrieve `n` elements starting at index `s` |

> [!TIP]
> To loop over **values**, use `for x in "${arr[@]}"`. To loop over **indices/keys**, use `for k in "${!arr[@]}"`. Always double-quote `"${arr[@]}"` so elements containing spaces are not word-split.

## Examples

### Indexed and associative arrays (`arrays.sh`)

```bash
#!/bin/bash

declare -a user_details

user_details[0]="1"

user_details[1]="rahul"

user_details[2]="jain"

user_details[3]="rahuljain5008@gmail.com"

echo "${user_details[@]}"

echo "User's ID: ${user_details[0]}"

echo "User's First Name : ${user_details[1]}"

echo "User's Last Name : ${user_details[2]}"

echo "${user_details[3]}"

declare -A user_details2

user_details2["id"]="1"

user_details2["first_name"]="rahul"

user_details2["last_name"]="jain"

user_details2["email"]="rahuljain5008@gmail.com"

echo "${user_details2[@]}"

echo "User's ID: ${user_details2["id"]}"

echo "User's First Name : ${user_details2["first_name"]}"

echo "User's Last Name : ${user_details2["last_name"]}"

echo "${user_details2["email"]}"
```

### Explicit indices and in-place updates (`arrays2.sh`)

```bash
#!/bin/bash

declare -a sport=(
[0]=football
[1]=cricket
[2]=hockey
[3]=basketball
)

echo "${sport[@]}"

echo "${sport[2]}"

echo "${sport[3]}"

sport[4]="dodgeball"

sport[2]="golf"

echo "${sport[@]}"

```

### Building an array from command output (`file.sh`)

Load all `.txt` files into an array and test each for the write permission bit.

```bash
#!/bin/bash

ARRAY=($(ls *.txt))
COUNT=0
echo -e "FILE NAME \t WRITEABLE"

for FILE in "${ARRAY[@]}"
do
  echo -n $FILE
  echo -n "[${#ARRAY[$COUNT]}]"
  if [ -w "$FILE" ]; then
    echo -e "\t YES"
  else
    echo -e "\t NO"
  fi

  let COUNT++
done
```

### Reading a file into an array (`readarray.sh`)

`readarray -t` (also spelled `mapfile`) reads a file line by line into an indexed array, stripping the trailing newline from each element.

```bash
#!/bin/bash

readarray -t FILE < /home/armour/Downloads/ip-list.txt

echo "KEY : Value"
for KEY in "${!FILE[@]}"; do
  # Print the KEY value
  echo "$KEY : ${FILE[$KEY]}"
done
```

### Iterating live-host results (`readarray-1.sh`)

```bash
#!/bin/bash

readarray -t ALL_LIVE_HOSTS < /tmp/nmap_outputs/LIVE-HOSTS.txt

for HOST in "${ALL_LIVE_HOSTS[@]}"
do
  echo "$HOST"
done


for HOST in "${!ALL_LIVE_HOSTS[@]}"
do
  echo "$HOST"
done
```

### Associative array as a command map (`commands.sh`)

An associative array can map a label to a command string, then execute each.

```bash
#!/bin/bash
LOG_FILE="/tmp/1.txt"
declare -A tests
tests["ls"]="ls 2>> $LOG_FILE &> /dev/null"
tests["ifconfig"]="ifconfig 2>> $LOG_FILE 1> /dev/null"
tests["cat"]="cat /etc/shadow 2>> $LOG_FILE 1> /dev/null"

i=0

for tool in "${!tests[@]}"; do
    i=$((i + 1))
    eval ${tests[$tool]}
done
```

## Best Practices

> [!TIP]
> - Prefer `readarray -t arr < file` over `arr=($(cat file))` — the former preserves lines with spaces and avoids glob expansion.
> - Always quote `"${arr[@]}"` in loops and expansions.
> - Use associative arrays for record-like data (`id`, `name`, `email`) instead of parallel indexed arrays that can drift out of sync.

## Security Considerations

> [!WARNING]
> The `commands.sh` pattern uses **`eval`** to run values pulled from the array. `eval` executes arbitrary shell and is a code-injection vector: if any array value derives from untrusted input (a file, an argument, network data), an attacker can inject commands. Only `eval` static, script-author-controlled strings — never data from outside the script. Where possible, invoke the command directly instead of building a string and `eval`-ing it.

- Building arrays from `ls`/glob output breaks on filenames containing spaces, newlines, or leading dashes. Prefer `readarray` with a null delimiter (`find ... -print0 | readarray -d ''`) for hostile filesystems.
- When arrays hold sensitive values (credentials, tokens), remember they are visible in the process memory and, if interpolated into commands, in `ps` output.

## Related

- [Shell-Scripting](Shell-Scripting.md) — parent guide for shell language constructs
- [Variables](Variables.md) — arrays are an extension of scalar variables
- [IF-Statement](IF-Statement.md) — iterating and testing array elements
- [Comparison-Operators](Comparison-Operators.md) — file tests used inside the array loops above
- [Linux Administration & Server Hardening](../Readme.md) — Linux administration & server hardening hub
