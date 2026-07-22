# Yum Command Reference

## Overview

A practical, task-oriented cheat sheet for the **Yum** (Yellowdog Updater, Modified) package manager on RPM-based distributions (CentOS, RHEL, Fedora). Commands are grouped by function — repository listing, search, install/update/remove, dependency and cache management, history, and package groups — followed by copy-paste examples.

On modern RHEL-family systems `yum` is a compatibility symlink to [DNF](DNF-Package-Manager.md), so every command below works unchanged there. For conceptual background and repository configuration, see [Yum(Yellowdog-Updater-Modified)](Yum(Yellowdog-Updater-Modified).md).

> [!TIP]
> Append `-y` to answer "yes" to all prompts for unattended/scripted runs, and `--skip-broken` to continue past packages with unresolved dependencies rather than aborting the whole transaction.

> [!IMPORTANT]
> Most `yum` operations that change system state require root. Prefer `sudo yum ...` over logging in as `root` so actions are attributable in the audit log.

## General Yum Commands

| Command | Description |
| :-- | :-- |
| `yum version` | Displays Yum version information. |
| `yum repolist` | Lists enabled repositories. |
| `yum repolist all` | Lists all repositories, including disabled ones. |
| `yum list` | Lists all available packages. |
| `yum list all` | Lists all packages (installed and available). |
| `yum list installed` | Lists installed packages. |
| `yum list available` | Lists packages available for installation. |
| `yum list recent` | Lists recently installed or updated packages. |
| `yum list extras` | Lists installed packages not available from enabled repos. |
| `yum check` | Checks for problems such as dependency issues. |
| `yum check-update` | Checks for available updates for installed packages. |
| `yum update` | Updates all installed packages to the latest version. |
| `yum upgrade` | Same as update but may also remove obsolete packages. |

## Package Search and Information

| Command | Description |
| :-- | :-- |
| `yum search <package>` | Search for packages containing the keyword. |
| `yum info <package>` | Displays detailed information about a package. |
| `yum provides <file package>` | Shows which package provides a specific file or feature. |


## Package Installation, Update, and Removal

| Command | Description |
| :-- | :-- |
| `yum install <package>` | Installs one or more packages. |
| `yum update <package>` | Updates a specific package. |
| `yum upgrade <package>` | Upgrades a specific package. |
| `yum remove <package>` | Removes/uninstalls a package. |
| `yum reinstall <package>` | Reinstalls an installed package. |
| `yum downgrade <package>` | Downgrades a package to an earlier version. |


## Package Dependency Management

| Command | Description |
| :-- | :-- |
| `yum deplist <package>` | Lists dependencies required by a package. |
| `yum provides <file package>` | Shows which package provides a specific file/feature. |

## Cache Management

| Command | Description |
| :-- | :-- |
| `yum clean all` | Removes all cached package data and metadata. |
| `yum makecache` | Generates and updates metadata cache. |
| `rm -rf /var/cache/dnf` | Manually deletes DNF cache directory (on newer systems). |


## Yum Package History and Sync

| Command | Description |
| :-- | :-- |
| `yum history` | Shows the transaction history of package operations. |
| `yum distro-sync` | Sync packages to the exact versions in enabled repos. |


## Yum Package Groups Management

| Command | Description |
| :-- | :-- |
| `yum groups` | List available package groups. |
| `yum groups list` | Same as `yum groups`. |
| `yum groups info "<group>"` | Display information about a package group. |
| `yum groups install "<group>"` | Install a package group. |
| `yum groups remove "<group>"` | Remove a package group. |
| `yum groups mark install "<group>"` | Mark a group as installed. |
| `yum groups mark remove "<group>"` | Mark a group as removed. |
| `yum groups summary "<group>"` | Show summary of a package group. |
| `yum groupinstall "<group>"` (CentOS 6) | Install package group (alternate syntax for CentOS 6). |
| `yum groupremove "<group>"` (CentOS 6) | Remove a package group (CentOS 6). |

## Examples of Useful Commands

- Check Yum version:

```bash
yum version
```

- List enabled repositories:

```bash
yum repolist
```

- List all repositories including disabled:

```bash
yum repolist all
```

- List all available packages:

```bash
yum list
```

- Count all packages:

```bash
yum list | wc -l
```

- List all packages (installed + available):

```bash
yum list all
```

- Count all packages (installed + available):

```bash
yum list all | wc -l
```

- List installed packages:

```bash
yum list installed
```

- Count installed packages:

```bash
yum list installed | wc -l
```

- List available packages:

```bash
yum list available
```

- Count available packages:

```bash
yum list available | wc -l
```

- List recently installed or updated packages:

```bash
yum list recent
```

- List installed packages not available from enabled repos:

```bash
yum list extras
```

- Check for problems and dependency issues:

```bash
yum check
```

- Check for available package updates:

```bash
yum check-update
```

- Update all installed packages:

```bash
yum update
```

- Upgrade all installed packages (with possible removals):

```bash
yum upgrade
```

- Search for a package by keyword:

```bash
yum search nmap
```

- Display detailed information about a package:

```bash
yum info gzip.x86_64
```

```bash
yum info python3
```
- Update a specific package:

```bash
yum update gzip
```

- Install Python:

```bash
yum install python
```

- Install curl:

```bash
yum install curl
```

- Install git:

```bash
yum install git
```

- Search for Vim packages:

```bash
yum search vim
```

```bash
yum info vim-enhanced.x86_64
```

```bash
yum install vim
```

- Install wget:

```bash
yum install wget
```

- Install all bash-related packages:

```bash
yum install bash-*
```

- Install net-tools package:

```bash
yum install net-tools.x86_64
```

- Install multiple packages together:

```bash
yum install wget bash-* net-tools nmap
```

- Install nmap:

```bash
yum install nmap
```

- Install all packages matching "nmap*":

```bash
yum install nmap*
```

- Remove package nmap:

```bash
yum remove nmap
```

- Remove all packages matching "nmap*":

```bash
yum remove nmap*
```

- Update package nmap:

```bash
yum update nmap
```

- Find which package provides nmap:

```bash
yum provides nmap
```

- Upgrade package nmap:

```bash
yum upgrade nmap
```

- Find which package provides vlc:

```bash
yum provides vlc
```

- Display info about Firefox:

```bash
yum info firefox
```

- Find which package provides Firefox:

```bash
yum provides firefox
```

- Update Firefox:

```bash
yum update firefox
```

- Search for VLC package:

```bash
yum search vlc
```

- Find which package provides VLC:

```bash
yum provides vlc
```

- Show VLC package info:

```bash
yum info vlc
```

- Install VLC:

```bash
yum install vlc
```

- Install VLC with automatic yes:

```bash
yum install vlc -y
```

- Upgrade Vim:

```bash
yum upgrade vim
```

- Downgrade VLC to previous version:

```bash
yum downgrade vlc
```

- Downgrade VLC skipping broken dependencies:

```bash
yum downgrade vlc --skip-broken
```

- List dependencies of VLC package:

```bash
yum deplist vlc
```

- Reinstall VLC package:

```bash
yum reinstall vlc
```

- Clean all Yum cached data:

```bash
yum clean all
```

- Update Yum metadata cache:

```bash
yum makecache
```

- Remove DNF cache manually (for newer systems):

```bash
cd /var/cache/dnf
```

```bash
rm -rf /var/cache/dnf
```

- Show Yum transaction history:

```bash
yum history
```

- List available languages for Yum:

```bash
yum langavailable
```

- Sync installed packages to repository versions:

```bash
yum distro-sync
```

### Yum Groups Commands

- List all package groups:

```bash
yum groups
```

- List all package groups (alternative):

```bash
yum groups list
```

- Show information about a group:

```bash
yum group info "System Tools"
```

- Install a package group:

```bash
yum groups install "System Tools"
```

- Install Development Tools group:

```bash
yum groups install "Development Tools"
```

- Remove a package group:

```bash
yum groups remove "system Tools"
```

- Summary of Compute Node group:

```bash
yum groups summary "system Tools"
```

- Mark group as installed:

```bash
yum groups mark install "Console Internet Tools"
```

- Mark group as removed:

```bash
yum groups mark remove "Console Internet Tools"
```

- Remove all GNOME-related packages:

```bash
yum remove gnome*
```




### CentOS 6 Specific Group Commands

- List package groups:

```bash
yum grouplist
```

- Install group in CentOS 6 format:

```bash
yum groupinstall "Virtualization Host"
```

- Install Haskell group:

```bash
yum groupinstall Haskell
```

- Remove Haskell group:

```bash
yum groupremove Haskell
```

## Best Practices

> [!TIP]
> - Run `yum check-update` before `yum update` to preview what will change.
> - Use `yum history` to review transactions and `yum history undo <id>` to roll back a bad update.
> - Avoid wildcard removals like `yum remove nmap*` on production hosts without first checking what matches — a broad glob can uninstall more than intended.
> - Clear stale metadata with `yum clean all` after changing repository files.

## Security Considerations

> [!IMPORTANT]
> - Keep `gpgcheck=1` on repositories so `yum` rejects unsigned or tampered packages.
> - Schedule `yum check-update` / `yum --security update` regularly and patch promptly — unpatched packages are a leading attack vector (CIS Benchmark control).
> - Review `yum provides` results and package sources before installing from third-party repositories.

## Related
- [Yum(Yellowdog-Updater-Modified)](Yum(Yellowdog-Updater-Modified).md) — yum concepts and usage
- [DNF-Package-Manager](DNF-Package-Manager.md) — yum's modern replacement
- [Red-Hat-Package-Manager(RPM)](Red-Hat-Package-Manager(RPM).md) — underlying .rpm package tool
- [Package-Manager-in-Linux](Package-Manager-in-Linux.md) — package management overview
- [Linux Administration & Server Hardening](../Readme.md) — Linux administration & server hardening hub
