# Enhance Bash with grc

## Overview

`grc` (Generic Colouriser) is a utility that adds colorized output to common command-line tools, making their output easier to read and interpret. It wraps a command, matches its output against per-tool regex configuration files, and applies ANSI colors — with no change to the underlying program. It works with commands such as `ping`, `netstat`, `traceroute`, `ps`, `df`, and many others.

This note walks through installing `grc` from source on a Red Hat-family system, wiring it into Bash so common commands are colorized automatically, and removing it cleanly.

> [!NOTE]
> `grc` only changes how output is *displayed*. It does not alter exit codes, stdout content when piped, or the behavior of the wrapped command — so it is safe to alias over everyday tools.

## Architecture

`grc` runs the real command in a pseudo-terminal, captures its output, and colorizes each line according to the matching configuration file before printing it to your terminal.

```mermaid
graph LR
    A[User runs: ping 8.8.8.8] --> B["Bash alias → grc ping 8.8.8.8"]
    B --> C[grc executes real ping]
    C --> D[Raw output lines]
    D --> E["grc matches conf regexes (grcat)"]
    E --> F[Colorized output to terminal]
```

## Configuration — Install Required Packages

- Install Bash completion support:

```bash
yum install bash-completion.noarch
```

- Install colored prompt support (if available in your repository):

```bash
yum install bash-color-prompt.noarch
```

- Install Python 3 (`grc` is a Python program):

```bash
yum install python3
```

- Install `wget`:

```bash
yum install wget
```

> [!TIP]
> For RHEL 8/9, Rocky Linux, AlmaLinux, and CentOS Stream, `dnf` may be used instead of `yum` — the two are compatible front-ends.

## Configuration — Install grc from Source

> Official project: https://github.com/garabik/grc

- Download grc:

```bash
wget https://github.com/garabik/grc/archive/refs/tags/v1.13.tar.gz
```

- Extract the archive:

```bash
tar -xvf v1.13.tar.gz
```

- Move to `/opt`:

```bash
mv -v grc-1.13 /opt/grc-1.13
```

- Navigate to the installation directory:

```bash
cd /opt/grc-1.13
```

- Install grc:

```bash
./install.sh
```

- Test grc by running a command through it:

```bash
grc ping google.com
```

> Example:

```bash
grc ping 8.8.8.8
```

- Run the original command without colorization:

```bash
ping 8.8.8.8
```

## Configuration — Enable grc Automatically in Bash

- Edit the Bash configuration file:

```bash
vim ~/.bashrc
```

Add the following configuration:

```bash
# Safe interactive aliases
alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'

# Enable grc aliases
GRC_ALIASES=true

# Load grc configuration if present
[[ -s "/etc/grc.sh" ]] && source /etc/grc.sh

# Load system-wide bash configuration
if [ -f /etc/bashrc ]; then
    . /etc/bashrc
fi
```

- Copy the grc configuration script (if the installer did not place it automatically):

```bash
cp -v /opt/grc-1.13/grc.sh /etc/
```

- Verify the file exists:

```bash
ls -l /etc/grc.sh
```

### Reload Bash Configuration

- Apply the changes without logging out:

```bash
source ~/.bashrc
```

### Verify grc Aliases

- Check whether aliases have been loaded:

```bash
alias | grep grc
```

> Example output:

```text
alias ping='grc ping'
alias traceroute='grc traceroute'
alias netstat='grc netstat'
```

> [!NOTE]
> **📸 Screenshot**
> _Capture: Terminal comparing plain ping output with grc ping output, where round-trip times and reply lines appear in distinct colors_

## Commands — Common Commands with grc

| Command | Purpose |
|---|---|
| `grc ping 8.8.8.8` | Colorize ICMP echo replies and round-trip times |
| `grc ps aux` | Colorize the process listing |
| `grc df -h` | Colorize disk usage by mount point |
| `grc netstat -rn` | Colorize the routing table |
| `grc traceroute google.com` | Colorize each hop and latency |

- Colorize `ping` output:

```bash
grc ping 8.8.8.8
```

- Colorize process listings:

```bash
grc ps aux
```

- Colorize disk usage:

```bash
grc df -h
```

- Colorize routing information:

```bash
grc netstat -rn
```

- Colorize traceroute output:

```bash
grc traceroute google.com
```

## Commands — Disable grc Temporarily

- Run the original command by escaping the alias:

```bash
\ping 8.8.8.8
```

- or bypass aliases with the `command` builtin:

```bash
command ping 8.8.8.8
```

## Configuration — Remove grc

- Remove the installation directory:

```bash
rm -rf /opt/grc-1.13
```

- Remove the configuration script:

```bash
rm -f /etc/grc.sh
```

- Remove grc entries from:

```bash
~/.bashrc
```

- Reload Bash:

```bash
source ~/.bashrc
```

## Best Practices

- **Keep the raw command reachable.** The `\command` and `command` escapes let you get uncolorized output for scripting or copy-paste; do not rely on colorized output being piped (grc detects non-TTY output and typically passes it through unchanged).
- **Prefer `GRC_ALIASES=true` over hand-writing aliases** so the alias set stays in sync with the shipped `grc.sh`.
- **Install from a pinned release tag** (as shown, `v1.13`) rather than `master` so your build is reproducible.

## Security Considerations

- **Source integrity:** you are installing from a downloaded tarball. On a hardened host, verify the download against the project's published checksum/signature before running `./install.sh`, since the installer runs as root and writes into `/etc` and system paths.
- **`grc` is a wrapper, not a sandbox.** It still executes the real command with your privileges; aliasing sensitive commands through it does not add or remove any access control.
- **Review `/etc/grc.sh` before sourcing it system-wide.** Anything sourced into every interactive Bash session is a persistence surface — treat changes to it like changes to `/etc/profile.d`.

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| `grc: command not found` | `install.sh` did not complete or `/usr/bin` not in `PATH` | Re-run the installer as root; check `PATH` |
| Aliases absent after reload | `/etc/grc.sh` missing or not sourced | Copy `grc.sh` to `/etc/`, confirm the `source` line in `~/.bashrc` |
| Output not colorized | Command has no matching grc conf, or output is piped | Only supported tools are colorized; grc passes non-TTY output through |
| Colors look wrong | `TERM` does not support 256 colors | Set a capable `TERM` (e.g. `xterm-256color`) |

## References

- Official project: https://github.com/garabik/grc
- `man 1 grc`, `man 1 grcat`

## Related

- [Shells-in-Linux](Shells-in-Linux.md) — shell overview and configuration.
- [Zsh-Shell](Zsh-Shell.md) — sources `/etc/grc.zsh` for the same colorization in Zsh.
- [Fish-Shell](Fish-Shell.md) — enables grc via `grc.fish` in `config.fish`.
- [Linux-Environment-Variables](Linux-Environment-Variables.md) — `GRC_ALIASES` and other env-driven settings.
- [Linux Administration & Server Hardening](../Readme.md) — Linux administration & server hardening hub.
