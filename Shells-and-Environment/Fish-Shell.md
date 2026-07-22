# Fish Shell

## Overview

Fish (Friendly Interactive Shell) is a modern command-line shell designed to be user-friendly and feature-rich out of the box. Unlike Bash or Zsh, it ships with syntax highlighting, history-based autosuggestions, rich tab completion, and a simplified scripting syntax without requiring plugins or extensive configuration.

This note covers installing Fish on a CentOS/RHEL-family system, making it the default shell, configuring it, enabling `grc` colorization inside Fish, and reverting to Bash.

> [!WARNING]
> Fish is **not** POSIX-compliant. Its scripting syntax (`set` instead of `export`, no `$(...)` command-substitution-in-quotes semantics, different `if`/`for` blocks) differs from Bash. Do **not** set Fish as the shell for service accounts or use it to run `#!/bin/sh` scripts; keep it for interactive human use.

## Configuration — Add the Fish Repository

> Project home: https://fishshell.com/

- Navigate to the YUM repository directory:

```bash
cd /etc/yum.repos.d/
```

- Download the Fish Shell repository file:

```bash
wget https://download.opensuse.org/repositories/shells:fish:release:3/CentOS_9_Stream/shells:fish:release:3.repo
```

## Configuration — Install Fish Shell

- Display configured repositories:

```bash
yum repolist all
```

- Install Fish:

```bash
yum install fish
```

## Commands — Verify Installation

- Display the Fish version:

```bash
fish --version
```

- Locate the Fish binary:

```bash
which fish
```

- Verify that Fish is listed as a valid login shell:

```bash
cat /etc/shells
```

> If required, add Fish to the list of valid shells:

```bash
echo $(which fish) >> /etc/shells
```

> [!IMPORTANT]
> `chsh` refuses any shell whose path is not present in `/etc/shells`. Add Fish to that file *before* attempting to change the default shell, or the change will be rejected.

## Configuration — Change the Default Shell to Fish

- Set Fish as the current user's login shell:

```bash
chsh -s $(which fish)
```

> Log out and log back in for the change to take effect.

## Commands — Verify the Shell Change

- Check the current login shell:

```bash
echo $SHELL
```

- Display the user's entry from the passwd database:

```bash
grep "^$(whoami):" /etc/passwd
```

## Commands — Start Fish Without Logging Out

```bash
fish
```

## Configuration — Fish Configuration

- User-specific Fish configuration file:

```bash
~/.config/fish/config.fish
```

- Create the configuration directory if it does not exist:

```bash
mkdir -p ~/.config/fish
```

- Edit the configuration file:

```bash
vim ~/.config/fish/config.fish
```

- Apply configuration changes:

```bash
source ~/.config/fish/config.fish
```

> [!NOTE]
> In Fish, environment variables are set with `set -x NAME value` (exported) or `set NAME value` (local), not with Bash's `export NAME=value`. Universal variables set with `set -U` persist across sessions automatically.

## Configuration — Enable grc in Fish Shell

See [Enhance-Bash-with-grc](Enhance-Bash-with-grc.md) for the full grc installation. Once grc is installed, wire it into Fish:

- Copy the grc Fish configuration:

```bash
cp -v /opt/grc-1.13/grc.fish /usr/local/etc/
```

- Verify the user configuration file exists:

```bash
ls -lh ~/.config/fish/config.fish
```

- Edit the Fish configuration:

```bash
vim ~/.config/fish/config.fish
```

> Add the following content:

```fish
if status is-interactive
    source /usr/local/etc/grc.fish
end
```

- Reload the configuration:

```fish
source ~/.config/fish/config.fish
```

## Commands — Useful Fish Features

| Feature | How to use | Notes |
|---|---|---|
| Autosuggestions | Type, then press Right Arrow | Suggests from history as you type |
| History search | `Ctrl + R` | Interactive reverse search |
| Tab completion | `Tab` | Shows completions with descriptions |
| Syntax highlighting | Automatic | Invalid commands shown in red |

### Command Autosuggestions

Fish automatically suggests commands based on history as you type.

- Accept a suggestion:

```text
Right Arrow
```

### Command History

- View command history:

```fish
history
```

- Search command history interactively:

```text
Ctrl + R
```

### Tab Completion

- Press:

```text
Tab
```

> Fish displays available completions with descriptions.

> [!NOTE]
> **📸 Screenshot**
> _Capture: Fish shell prompt showing a greyed-out autosuggestion after a partially typed command and red highlighting on an invalid command name_

## Configuration — Revert Back to Bash

- Locate the Bash binary:

```bash
which bash
```

- Change the default shell back to Bash:

```bash
chsh -s $(which bash)
```

## Best Practices

- **Keep Fish for interactive use only.** Author automation in POSIX `sh`/Bash so it runs under cron, systemd, and other hosts.
- **Use universal variables (`set -U`) for persistent interactive settings** instead of editing `config.fish` by hand where possible — they survive upgrades cleanly.
- **Guard interactive-only setup with `status is-interactive`** (as shown for grc) so non-interactive Fish invocations start fast and cleanly.

## Security Considerations

- **Do not assign Fish to service or system accounts.** Non-POSIX behavior can break login scripts and hardening tooling that assumes a Bourne-compatible shell; service accounts should use `/usr/sbin/nologin`.
- **Review third-party sources before sourcing them into `config.fish`.** Anything sourced there runs at the start of every interactive session and is a persistence surface.
- **Repository trust:** the Fish package here comes from the openSUSE build service repo. On a hardened host, confirm the repo's GPG key and pin the release before enabling it.

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| `chsh: shell not found in /etc/shells` | Fish path not in `/etc/shells` | `echo $(which fish) >> /etc/shells`, retry |
| Bash one-liner fails in Fish | Non-POSIX syntax | Run it explicitly with `bash -c '...'` |
| `export: command not found` | Using Bash syntax in Fish | Use `set -x NAME value` instead |
| grc colors absent in Fish | `grc.fish` not sourced | Confirm the `source /usr/local/etc/grc.fish` block in `config.fish` |

## References

- Project home & docs: https://fishshell.com/
- `man 1 fish`, `man 1 fish_config`

## Related

- [Shells-in-Linux](Shells-in-Linux.md) — shell overview and configuration.
- [Zsh-Shell](Zsh-Shell.md) — comparable modern, feature-rich shell.
- [Enhance-Bash-with-grc](Enhance-Bash-with-grc.md) — install grc, then source `grc.fish` here.
- [Linux-Environment-Variables](Linux-Environment-Variables.md) — variable handling differs (`set -x` vs `export`).
- [Linux Administration & Server Hardening](../Readme.md) — Linux administration & server hardening hub.
