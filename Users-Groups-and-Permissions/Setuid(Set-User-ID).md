# Setuid (Set User ID)

## Overview

**SUID** (Set User ID) is a special file permission that causes an executable to run with the privileges of its **file owner** (often `root`), regardless of who launches it. It is what allows unprivileged users to perform tightly-scoped privileged actions safely.

The classic example is `passwd`: an ordinary user can change their own password because `passwd` is SUID-root, giving it the temporary access it needs to edit protected files such as `/etc/shadow` — without granting the user a general root shell.

SUID is shown as an `s` in the **owner execute** position of an `ls -l` listing and by the octal digit **`4`** in the special-permissions field.

> [!IMPORTANT]
> SUID concentrates power in a single binary. A safe, well-audited SUID program does exactly one privileged thing; a misconfigured or attacker-planted one is a direct path to full system compromise. Treat every SUID binary as security-critical.

## Concepts

### The Special-Permission Bits

The leading (fourth) octal digit in `chmod` selects the special permission bits:

| Octal | Bit | Effect |
| :-- | :-- | :-- |
| `4` | setuid | Run with the file owner's UID |
| `2` | setgid | Run with the file's group ID |
| `1` | sticky | Restrict deletion in shared directories |

### How SUID Changes Identity

When a non-SUID program runs, its effective UID equals the caller's UID. When an SUID program runs, the kernel sets the process's **effective UID** to the file owner's UID, so file-access checks are performed as the owner while the program executes.

## Commands

The `fdisk` binary is a useful demonstration target: listing partitions normally requires root.

- Locate the `fdisk` binary — find the absolute path of the executable so its permissions can be manipulated:

```bash
which fdisk
```

- Display disk partition information — by default this requires root or SUID-root privileges:

```bash
fdisk -l
```

- Create a new user with root privileges — this creates a user named `root1` with UID 0 (root's user ID). This is inherently dangerous and should only be used for demonstration:

```bash
useradd root1 -o -u 0
```

- Grant SUID to `fdisk` — enables anyone to execute `fdisk` with root privileges:

```bash
chmod u+s /usr/sbin/fdisk
```

- Verify new permissions — after setting SUID, the owner's execute bit (`x`) is replaced by `s`:

```bash
ls -lh /usr/sbin/fdisk
```

- Run `fdisk` as a regular user — after SUID is set, even non-root users can execute `fdisk` as root:

```bash
fdisk -l
```

- Remove SUID from `fdisk` — for safety, revert permissions to disable setuid:

```bash
chmod u-s /usr/sbin/fdisk
```

- Set permissions and SUID together (octal notation) — combines permission and SUID in a single step (`4` = SUID):

```bash
chmod 4755 /usr/sbin/fdisk
```

- Restore standard permissions (remove SUID) — removes setuid while maintaining normal execute permissions:

```bash
chmod 0755 /usr/sbin/fdisk
```

> [!WARNING]
> Creating a UID 0 user (`useradd -o -u 0`) and adding SUID to `/usr/sbin/fdisk` are demonstrations of *how the mechanism works*. Never leave either in place on a real system — both grant trivial root access.

## Examples

### Custom Setuid Root Shell

This end-to-end example shows why an SUID-root, user-supplied binary is so dangerous: it hands the caller a full root shell.

- Create C code for a root shell — this program attempts to escalate privileges and launch a root shell:

```bash
vim rootshell.c
```

```c
#include <stdio.h>
#include <unistd.h>

int main(void) {
    // Elevate group IDs first, then user IDs
    setgid(0);
    setegid(0);
    setuid(0);
    seteuid(0);

    // Properly format the arguments array for execvp
    char *args[] = {"/bin/sh", NULL};

    // Execute the shell
    execvp(args[0], args);

    // execvp only returns if an error occurs
    perror("execvp failed");
    return 1;
}
```

- Install the C compiler (if not present):

```bash
yum install gcc
```

- Compile the root shell program:

```bash
gcc rootshell.c -o rootshell
```

- Enable SUID on the compiled binary — allows the program to run as root regardless of the user:

```bash
chmod u+s rootshell
```

- Run your setuid-root shell — you now have root access as an ordinary user (assuming no system-level SUID protections):

```bash
./rootshell
```

> [!WARNING]
> **Security Warning**
> Granting SUID to executables can allow unintended privilege escalation and exposes the system to severe risk if misused or left on unauthorised binaries. Always restrict SUID use to trusted, essential system binaries, and **never** set SUID on interpreters or user-created programs outside a secure, isolated lab.

### Inspecting `passwd` and Sensitive Files

- Check sensitive file permissions — observe who can read privileged files (typically only root):

```bash
ls -lh /etc/shadow
```

```bash
ls -lh /etc/gshadow
```

- See how the `passwd` command uses SUID — the `s` in owner-execute denotes SUID, which lets ordinary users change their passwords by giving the program root-level access to `/etc/shadow`:

```bash
which passwd
```

```bash
ls -lh /usr/bin/passwd
```

- Remove SUID from `passwd` (not recommended) — this prevents standard users from updating their passwords:

```bash
chmod u-s /usr/bin/passwd
```

## Concepts: `/usr/sbin` vs `/usr/bin`

`/usr/sbin` and `/usr/bin` both hold executable binaries, but they are separated by *who* the programs are meant for:

- **`/usr/bin`** — user commands and executables intended for general use by all users. These are typical applications (editors like `vim`, utilities like `scp`) that do *not* require administrative privileges to run.
- **`/usr/sbin`** — system-administration binaries intended primarily for the system administrator (root or users with sudo). Examples include `fdisk`, `ifconfig`, and `shutdown`, which are generally not needed (or allowed) for ordinary users.

### Key Differences

| Directory | Intended For | Typical Use Cases | Example Programs |
| :-- | :-- | :-- | :-- |
| `/usr/bin` | All users | Regular applications | `vim`, `du`, `diff` |
| `/usr/sbin` | Admin / root users | System / admin tools | `fdisk`, `ifconfig` |

### Context and Evolution

- Historically, admin tools were kept separate for clarity and to help restrict access, but this is largely an organisational convention, not a security mechanism.
- On modern Linux systems there is often little technical restriction: ordinary users simply won't have permission to *use* tools in `/usr/sbin` that require system privileges, but the directory itself is accessible.
- On many distributions `/bin` is now a symlink to `/usr/bin` and `/sbin` to `/usr/sbin`. The separation is now mostly traditional rather than essential.

## Best Practices

- Keep the SUID inventory minimal — every SUID binary is attack surface. Remove SUID from anything that does not strictly need it.
- Audit regularly with `find / -perm -4000 -type f 2>/dev/null` and confirm each result is a known, package-provided binary.
- Never apply SUID to shells, interpreters, or custom scripts.
- Mount user-writable filesystems with `nosuid` so SUID bits there are ignored.
- Use `sudo` with fine-grained rules for delegated administration instead of hand-rolled SUID wrappers.

## Security Considerations

> [!WARNING]
> SUID-root binaries are a primary Linux privilege-escalation vector. Attackers enumerate them (`find / -perm -4000`) and search projects like GTFOBins for known-abusable entries.

- Maintain a baseline of expected SUID binaries and alert on any addition — this aligns with CIS Benchmark and NIST file-integrity guidance.
- Prefer Linux **capabilities** (`setcap`) over full SUID-root where a program needs only one privilege (for example `cap_net_raw` for `ping`).
- Removing SUID from `passwd` breaks unprivileged password changes — a reminder that SUID is sometimes essential, so audit rather than blanket-strip.

## Troubleshooting

| Symptom | Likely cause | Fix |
| :-- | :-- | :-- |
| SUID shows as capital `S` in `ls -l` | Owner execute bit is missing | Add owner execute: `chmod u+x <file>` (or use `4755`). |
| SUID binary "does nothing" as a non-root user | Filesystem mounted `nosuid` | Check `mount` output; the bit is intentionally ignored there. |
| SUID script has no effect | Modern kernels ignore SUID on scripts | Compile a small binary wrapper or use `sudo` instead. |
| Users can no longer change passwords | SUID removed from `/usr/bin/passwd` | Restore it: `chmod u+s /usr/bin/passwd`. |

## Related

- [Setgid(Set-Group-ID)](Setgid(Set-Group-ID).md) — counterpart group-ID bit.
- [Sticky-Bit](Sticky-Bit.md) — companion special permission bit.
- [Special-Permission](Special-Permission.md) — overview of setuid, setgid, and the sticky bit.
- [Find-Command](../String-Processing-and-Finding-Files/Find-Command.md) — use `find -perm` to locate SUID binaries.
- Privilege-Escalation — SUID binaries as a privilege-escalation vector.
- [Linux Administration & Server Hardening](../Readme.md) — course hub.
