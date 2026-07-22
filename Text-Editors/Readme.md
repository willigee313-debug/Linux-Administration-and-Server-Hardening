# Text Editors

Editing configuration and scripts efficiently with Nano and Vi/Vim.

> [!NOTE]
> **Module hub**
> Part of the **[Linux Administration & Server Hardening](../Readme.md)** course.

## Overview

Server administration is largely editing text: config files, unit files, and scripts. This module covers Nano for quick edits and Vi/Vim for powerful modal editing — including modes, search-and-replace, macros, and a maintainable vimrc — so you can work fluently on any system, even a minimal one with no GUI.

## Learning Objectives

By the end of this module you will be able to:

- Edit and save files confidently in Nano and Vim over SSH
- Use Vim modes, motions, search-and-replace, and macros to edit quickly
- Maintain a portable vimrc and apply editor productivity techniques

## Topics Covered

This module contains **10 notes**.

| Note | Topic |
| --- | --- |
| [Editor-Productivity-Tips](Editor-Productivity-Tips.md) | Editor Productivity Tips |
| [Introduction-to-Text-Editors](Introduction-to-Text-Editors.md) | Introduction to Text Editors |
| [Nano-Command](Nano-Command.md) | Nano Command |
| [Nano-Editor](Nano-Editor.md) | Nano Editor |
| [Search-and-Replace-in-Vim](Search-and-Replace-in-Vim.md) | Search and Replace in Vim |
| [Vi-and-Vim-Editor](Vi-and-Vim-Editor.md) | Vi and Vim Editor |
| [Vim-Command](Vim-Command.md) | Vim Command |
| [Vim-Configuration-vimrc](Vim-Configuration-vimrc.md) | Vim Configuration vimrc |
| [Vim-Macros](Vim-Macros.md) | Vim Macros |
| [Vim-Modes](Vim-Modes.md) | Vim Modes |

## Practical Labs

> [!NOTE]
> **Hands-on labs**
> Guided, reproducible labs for this topic live in the course-wide **[Practical Labs](../Practical-Labs/Readme.md)** collection — build them on a disposable VM. The step-by-step configuration walkthroughs in the notes above also work as self-paced labs.

## Best Practices

- Learn one modal editor well — Vim is present on virtually every system
- Keep a minimal, commented vimrc under version control
- Make a timestamped backup before editing critical config by hand

## Security Considerations

> [!WARNING]
> **Harden before you expose**
> Apply these controls before placing any service on an untrusted network.

- Never paste secrets into an editor buffer that may be swap-cached; disable swap files for sensitive edits (`:set noswapfile`)
- Validate service config with the daemon's own checker before reloading

## Troubleshooting

| Symptom | Likely cause & fix |
| --- | --- |
| Stuck in Vim and cannot exit | Press `Esc` then type `:q!` to quit without saving |
| Edits to a config had no effect | Confirm you edited the active file and reloaded/restarted the service |

## References

- [Vim documentation](https://www.vim.org/docs.php)
- [GNU Nano manual](https://www.nano-editor.org/docs.php)

## Related Notes

- [Linux Administration & Server Hardening](../Readme.md) — course hub and map of content
- [Shell Scripting](../Shell-Scripting/Readme.md) — related module
- [Shells and Environment](../Shells-and-Environment/Readme.md) — related module
