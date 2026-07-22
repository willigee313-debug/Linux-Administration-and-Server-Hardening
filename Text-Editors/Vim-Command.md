# Vim Command

`vim` is a powerful and widely used text editor on Unix/Linux systems. It stands for **Vi IMproved** — an enhanced, backward-compatible version of the older `vi` editor.

## Overview

- `vi` comes from "Visual Editor".
- Vim is **not** WYSIWYG (What You See Is What You Get) — it is a modal, keyboard-driven editor.
- It is used for editing configuration files, writing scripts, and programming.

Vim is installed by default on the vast majority of Linux and BSD systems, which makes it the reliable fallback editor for any remote or minimal environment. Its efficiency comes from **modes**: the same keys mean different things depending on the mode you are in, so nearly every edit can be performed without leaving the keyboard's home row.

> [!NOTE]
> **📸 Screenshot**
> _Capture: Vim editing testfile.txt in a terminal, with line numbers on the left, the text buffer in the centre, and the status line at the bottom showing the mode indicator and cursor position_

## Concepts

### Vim Modes

Vim operates in several modes, each with a specific purpose:

| Mode | Enter With | Purpose |
|---|---|---|
| **Command (Normal) Mode** | `Esc` (default on start) | Move, delete, copy, and navigate text. |
| **Insert Mode** | `i` or `insert` | Type and insert text. |
| **Visual Mode** | `v` | Select blocks of text. |
| **Replace Mode** | `R` | Overwrite existing characters. |
| **Command-Line (Last Line) Mode** | `:` | Save, quit, and run search/replace commands. |

The relationship between the modes is easiest to see as a state diagram — `Esc` always returns you to Command mode:

```mermaid
stateDiagram-v2
    [*] --> Command
    Command --> Insert: i / insert
    Command --> Visual: v
    Command --> Replace: R
    Command --> CommandLine: :
    Insert --> Command: Esc
    Visual --> Command: Esc
    Replace --> Command: Esc
    CommandLine --> Command: Enter / Esc
```

> [!TIP]
> **When in doubt about which mode you are in, press `Esc` to return to Command mode, then continue.**

## Commands

### Launch Vim

- Open the `vim` editor without any file.

```bash
vim
```

- Open the Vim help menu.

```bash
:help
```

- Open or create a file named `filename`.

```bash
vim filename
```

### Moving the Cursor

Movement keys are pressed in Command mode. Prefix them with a count to repeat.

| Key | Movement |
|---|---|
| `h` | Move left by one character |
| `j` | Move down one line |
| `k` | Move up one line |
| `l` | Move right one character |

### Deleting Characters

| Command | Action |
|---|---|
| `x` | Delete a single character |
| `10x` | Delete 10 characters |
| `X` | Delete character before the cursor |
| `10X` | Delete 10 characters before the cursor |

### Deleting Words

| Command | Action |
|---|---|
| `dw` | Delete one word from the cursor |
| `10dw` | Delete 10 words |

### Deleting Lines

| Command | Action |
|---|---|
| `dd` | Delete one line |
| `10dd` | Delete 10 lines |
| `D` | Delete to the end of the line from the cursor |

### Replacing Characters

| Command | Action |
|---|---|
| `r` | Replace a single character |
| `3r` | Replace 3 characters |

### Replacing Words

| Command | Action |
|---|---|
| `cw` | Change (replace) a word from the cursor |
| `3cw` | Change 3 words |

### Replacing Lines

| Command | Action |
|---|---|
| `C` | Change text from the cursor to the end of line |
| `3C` | Replace 3 lines |

### Insert Blank Lines

| Command | Action |
|---|---|
| `o` | Insert a blank line **below** the current line |
| `10o` | Insert 10 blank lines below |
| `O` | Insert a blank line **above** the current line |
| `10O` | Insert 10 blank lines above |

### Joining Lines

| Command | Action |
|---|---|
| `J` | Join two lines into one |
| `3J` | Join three lines into one |

### Copy and Paste Lines

| Command | Action |
|---|---|
| `yy` | Copy (yank) a line |
| `4yy` | Copy 4 lines |
| `p` | Paste a copied line |

### Command-Line Mode

Command-Line (Last Line) mode is entered with `:` and is used for file operations, settings, and search/replace.

- Run shell commands from within Vim:

```bash
:!ls
```

```bash
:!pwd
```

```bash
:!id
```

- Enable line numbers:

```bash
:set number
```

```bash
:set nu
```

- Disable line numbers:

```bash
:set nonumber
```

```bash
:set nonu
```

## Examples

### Sample Test Content (testfile.txt)

- Create the test file using Vim:

```bash
vim testfile.txt
```

> Paste the following content into the file:

```text
Hello, this is AI-generated content. for test AI
AI is changing the world.
Armour Infosec specializes in AI and Cyber Security.
AI is powerful.
This line is blank below:


Change /bin/bash to /sbin/nologin for restricted users.
Some users still use /bin/bash.
Change bash to sh if needed.
Line Numbering helps in editing.

This file is for testing search and replace in Vim.
Another AI-powered sentence. AI

End of AI content.
```

### Search and Replace Examples

The `:s` (substitute) command is Vim's find-and-replace engine. The general form is `:[range]s/pattern/replacement/[flags]`, where `%` means the whole file, `g` replaces all matches on a line, `c` prompts for confirmation, and `i` makes the match case-insensitive.

- Replace `AI` with `Artificial Intelligence` globally:

```vim
:%s/AI/Artificial Intelligence/g
```

- Replace `bash` with `sh` with a confirmation prompt:

```vim
:%s/bash/sh/gc
```

- Replace only `/bin/bash` with `/sbin/nologin` (slashes escaped):

```vim
:%s/\/bin\/bash/\/sbin\/nologin/g
```

- Replace `AI` with `ML` from line 3 to line 6:

```vim
:3,6s/AI/ML/g
```

- Remove extra blank lines:

```vim
:g/^$/d
```

- Replace within the current line only:

```vim
:s/AI/Artificial Intelligence/
```

- Replace between line 1 and line 32:

```vim
:1,32s/AI/Artificial Intelligence
```

- Prompt before each replacement globally:

```vim
:%s/AI/Artificial Intelligence/c
```

- Global search and replace, `nologin` to `bash`:

```vim
:%s/nologin/bash/g
```

- Case-sensitive replacement with confirmation:

```vim
:%s/bash/sh/gc
```

- Case-insensitive replacement, `Bash` to `sh`:

```vim
:%s/Bash/sh/gi
```

- Replace in a specific line range, 56 to 65:

```vim
:56,65s/bahs/bash/g
```

- Change the default login shell for all users:

```vim
:%s/\/bin\/bash/\/sbin\/nologin/g
```

- Revert the shell change:

```vim
:%s/\/sbin\/nologin/\/bin\/bash/g
```

- Clear search highlighting:

```vim
:nohl
```

## Best Practices

- **Learn the mode discipline first.** Reflexively pressing `Esc` before issuing a command avoids the classic mistake of typing commands into the buffer as text.
- **Use ranges deliberately.** `%` operates on the whole file; a numeric range like `3,6` limits blast radius. When editing production config, prefer the smallest range that does the job.
- **Add the `c` flag for risky substitutions** so Vim prompts before each change, letting you skip unintended matches.
- **Persist your preferences** (line numbers, syntax highlighting, indentation) in `~/.vimrc` instead of retyping `:set` commands every session.

## Security Considerations

> [!WARNING]
> **Running Vim as root (or via `sudo`) can be abused for privilege escalation: from Command-Line mode you can spawn a shell with `:!/bin/sh` or `:set shell=/bin/sh` then `:shell`. This is a documented GTFOBins technique for `vi`/`vim` with an SUID bit or a permissive `sudo` rule.**

- Editing sensitive files such as `/etc/passwd`, `/etc/shadow`, or `/etc/sudoers` with a mistaken substitution can lock users out or open an escalation path. For `/etc/sudoers`, prefer `visudo`, which validates syntax before writing.
- Vim's shell-escape feature (`:!command`) executes commands with the current user's privileges — be mindful of this on shared or restricted systems.
- Swap files (`.swp`) and undo files may persist partial contents of an edited file on disk; on hardened hosts, restrict directory permissions or disable them for sensitive edits.

## Troubleshooting

| Symptom | Likely Cause | Resolution |
|---|---|---|
| Stuck and cannot type text | You are in Command mode | Press `i` to enter Insert mode; press `Esc` to return to Command mode. |
| `E45: 'readonly' option is set` on save | File opened read-only or owned by root | Save elsewhere, or reopen with `sudo vim <file>`; force with `:w!` only if intended. |
| Cannot quit | Unsaved changes block `:q` | `:wq` to save and quit, or `:q!` to discard changes and quit. |
| "Swap file already exists" warning | A previous Vim session crashed | Recover with `:recover`, or delete the stale `.swp` file after verifying no other session is open. |
| Highlighted matches won't clear | Search highlighting still active | Run `:nohl`. |

## Vim Practice Tool

- Launch the built-in interactive tutorial:

```bash
vimtutor
```

> Practice Vim commands through a game:

[https://vim-adventures.com/](https://vim-adventures.com/)

## References

- `:help` inside Vim and the `vimtutor` interactive tutorial
- Vim documentation: <https://vimhelp.org/>
- GTFOBins (vim privilege-escalation reference): <https://gtfobins.github.io/gtfobins/vim/>

## Related
- [Nano-Command](Nano-Command.md) — beginner-friendly, modeless terminal editor
- [Vim-Modes](Vim-Modes.md) — deeper explanation of Vim's editing modes
- [Search-and-Replace-in-Vim](Search-and-Replace-in-Vim.md) — focused guide to the `:s` substitute command
- [Vim-Configuration-vimrc](Vim-Configuration-vimrc.md) — persisting settings in `~/.vimrc`
- [Vim-Macros](Vim-Macros.md) — recording and replaying keystroke macros
- [Linux-Basic-Commands](../Linux-Basic-Commands/Linux-Basic-Commands.md) — core command-line workflow
- Privilege-Escalation — vim via sudo/SUID is a GTFOBins privesc vector
- [Linux Administration & Server Hardening](../Readme.md) — course hub
