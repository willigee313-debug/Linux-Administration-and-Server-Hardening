# Zsh Shell

## Overview

Zsh (Z Shell) is an advanced Unix shell that is largely compatible with Bash while adding powerful interactive features: improved tab completion, sophisticated history management, spelling correction, extended globbing, themes, and a rich plugin ecosystem (commonly via the Oh My Zsh framework). It is the default interactive shell on Kali Linux and macOS.

This note covers installing Zsh, making it the default login shell, its core interactive features, installing Oh My Zsh, the reference Kali `.zshrc`, and reverting to Bash.

> [!NOTE]
> Zsh reads a different set of startup files from Bash: `~/.zshenv` (always), `~/.zprofile` (login), `~/.zshrc` (interactive), and `~/.zlogin` (login, after zshrc). Most user configuration lives in `~/.zshrc`.

## Configuration — Install Zsh

- Search for available Zsh packages:

```bash
yum search zsh
```

- Install Zsh:

```bash
yum install zsh
```

```bash
yum install zsh*
```

## Commands — Verify Installation

- Display the Zsh version:

```bash
zsh --version
```

- Locate the Zsh binary:

```bash
which zsh
```

## Configuration — Verify Valid Login Shells

- Display the list of valid login shells:

```bash
cat /etc/shells
```

> If the Zsh binary path is not listed, add it manually:

```bash
echo $(which zsh) >> /etc/shells
```

> [!IMPORTANT]
> `chsh` only accepts a shell whose absolute path appears in `/etc/shells`. Add the Zsh path there first if it is missing, or the default-shell change will fail.

## Configuration — Change the Default Shell to Zsh

- Change the current user's login shell:

```bash
chsh -s $(which zsh)
```

> Log out and log back in for the change to take effect.

## Commands — Verify the Shell Change

- Display the user's shell entry from the passwd database:

```bash
grep "^$(whoami):" /etc/passwd
```

- Check the currently configured login shell:

```bash
echo $SHELL
```

## Commands — Start Zsh Without Logging Out

```bash
zsh
```

## Configuration — Zsh Initial Configuration

- When Zsh is started for the first time, it may launch an interactive configuration wizard.

- To manually start the configuration wizard:

```bash
autoload -Uz zsh-newuser-install
```

```bash
zsh-newuser-install -f
```

## Configuration — Common Zsh Configuration File

- User-specific configuration:

```bash
~/.zshrc
```

- Edit the configuration file:

```bash
vi ~/.zshrc
```

- Apply configuration changes:

```bash
source ~/.zshrc
```

## Commands — Useful Zsh Features

| Feature | Trigger | Description |
|---|---|---|
| Auto-completion | `Tab` | Complete commands, filenames, and directories |
| History search | `Ctrl + R` | Interactive reverse search of prior commands |
| Spelling correction | `setopt CORRECT` | Suggests corrections for mistyped commands |
| Menu completion | `Tab` `Tab` | Cycle through candidates in a menu |

### Command Auto-Completion

- Press the `Tab` key to automatically complete commands, filenames, and directories.

### Command History Search

- Search previous commands interactively:

```bash
history
```

Use:

```text
Ctrl + R
```

### Command Correction

- Enable automatic spelling correction:

```bash
setopt CORRECT
```

- Add it permanently to `~/.zshrc`:

```bash
echo "setopt CORRECT" >> ~/.zshrc
```

## Configuration — Install Oh My Zsh

Oh My Zsh is a popular framework that provides themes and plugins for Zsh.

- Install Git if not already installed:

```bash
yum install git
```

- Install Oh My Zsh:

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

> [!WARNING]
> `curl | sh` executes a remote script with your privileges before you have inspected it. On a hardened or production host, download the installer first, review it, and verify its source before running.

## Configuration — Common Oh My Zsh Configuration

- Edit the Oh My Zsh configuration:

```bash
vim ~/.zshrc
```

- Change the theme:

```bash
ZSH_THEME="robbyrussell"
```

- Apply the changes:

```bash
source ~/.zshrc
```

## Configuration — Kali Linux .zshrc Reference

Kali ships a well-tuned `.zshrc` with a two-line prompt, syntax highlighting, autosuggestions, extensive keybindings, and history tuning. Open it for editing:

```bash
vim ~/.zshrc
```

The full reference configuration is reproduced below.

```bash
# ~/.zshrc file for zsh interactive shells.
# see /usr/share/doc/zsh/examples/zshrc for examples

setopt autocd              # change directory just by typing its name
#setopt correct            # auto correct mistakes
setopt interactivecomments # allow comments in interactive mode
setopt magicequalsubst     # enable filename expansion for arguments of the form ‘anything=expression’
setopt nonomatch           # hide error message if there is no match for the pattern
setopt notify              # report the status of background jobs immediately
setopt numericglobsort     # sort filenames numerically when it makes sense
setopt promptsubst         # enable command substitution in prompt

WORDCHARS=${WORDCHARS//\/} # Don't consider certain characters part of the word

# hide EOL sign ('%')
PROMPT_EOL_MARK=""

# configure key keybindings
bindkey -e                                        # emacs key bindings
bindkey ' ' magic-space                           # do history expansion on space
bindkey '^U' backward-kill-line                   # ctrl + U
bindkey '^[[3;5~' kill-word                       # ctrl + Supr
bindkey '^[[3~' delete-char                       # delete
bindkey '^[[1;5C' forward-word                    # ctrl + ->
bindkey '^[[1;5D' backward-word                   # ctrl + <-
bindkey '^[[5~' beginning-of-buffer-or-history    # page up
bindkey '^[[6~' end-of-buffer-or-history          # page down
bindkey '^[[H' beginning-of-line                  # home
bindkey '^[[F' end-of-line                        # end
bindkey '^[[Z' undo                               # shift + tab undo last action

# enable completion features
autoload -Uz compinit
compinit -d ~/.cache/zcompdump
zstyle ':completion:*:*:*:*:*' menu select
zstyle ':completion:*' auto-description 'specify: %d'
zstyle ':completion:*' completer _expand _complete
zstyle ':completion:*' format 'Completing %d'
zstyle ':completion:*' group-name ''
zstyle ':completion:*' list-colors ''
zstyle ':completion:*' list-prompt %SAt %p: Hit TAB for more, or the character to insert%s
zstyle ':completion:*' matcher-list 'm:{a-zA-Z}={A-Za-z}'
zstyle ':completion:*' rehash true
zstyle ':completion:*' select-prompt %SScrolling active: current selection at %p%s
zstyle ':completion:*' use-compctl false
zstyle ':completion:*' verbose true
zstyle ':completion:*:kill:*' command 'ps -u $USER -o pid,%cpu,tty,cputime,cmd'

# History configurations
HISTFILE=~/.zsh_history
HISTSIZE=1000
SAVEHIST=2000
setopt hist_expire_dups_first # delete duplicates first when HISTFILE size exceeds HISTSIZE
setopt hist_ignore_dups       # ignore duplicated commands history list
setopt hist_ignore_space      # ignore commands that start with space
setopt hist_verify            # show command with history expansion to user before running it
#setopt share_history         # share command history data

# force zsh to show the complete history
alias history="history 0"

# configure `time` format
TIMEFMT=$'\nreal\t%E\nuser\t%U\nsys\t%S\ncpu\t%P'

# make less more friendly for non-text input files, see lesspipe(1)
#[ -x /usr/bin/lesspipe ] && eval "$(SHELL=/bin/sh lesspipe)"

# set variable identifying the chroot you work in (used in the prompt below)
if [ -z "${debian_chroot:-}" ] && [ -r /etc/debian_chroot ]; then
    debian_chroot=$(cat /etc/debian_chroot)
fi

# set a fancy prompt (non-color, unless we know we "want" color)
case "$TERM" in
    xterm-color|*-256color) color_prompt=yes;;
esac

# uncomment for a colored prompt, if the terminal has the capability; turned
# off by default to not distract the user: the focus in a terminal window
# should be on the output of commands, not on the prompt
force_color_prompt=yes

if [ -n "$force_color_prompt" ]; then
    if [ -x /usr/bin/tput ] && tput setaf 1 >&/dev/null; then
        # We have color support; assume it's compliant with Ecma-48
        # (ISO/IEC-6429). (Lack of such support is extremely rare, and such
        # a case would tend to support setf rather than setaf.)
        color_prompt=yes
    else
        color_prompt=
    fi
fi

configure_prompt() {
    prompt_symbol=㉿
    # Skull emoji for root terminal
    #[ "$EUID" -eq 0 ] && prompt_symbol=
    case "$PROMPT_ALTERNATIVE" in
        twoline)
            PROMPT=$'%F{%(#.blue.green)}┌──${debian_chroot:+($debian_chroot)─}${VIRTUAL_ENV:+($(basename $VIRTUAL_ENV))─}(%B%F{%(#.red.blue)}%n'$prompt_symbol$'%m%b%F{%(#.blue.green)})-[%B%F{reset}%(6~.%-1~/…/%4~.%5~)%b%F{%(#.blue.green)}]\n└─%B%(#.%F{red}#.%F{blue}$)%b%F{reset} '
            # Right-side prompt with exit codes and background processes
            #RPROMPT=$'%(?.. %? %F{red}%B⨯%b%F{reset})%(1j. %j %F{yellow}%B%b%F{reset}.)'
            ;;
        oneline)
            PROMPT=$'${debian_chroot:+($debian_chroot)}${VIRTUAL_ENV:+($(basename $VIRTUAL_ENV))}%B%F{%(#.red.blue)}%n@%m%b%F{reset}:%B%F{%(#.blue.green)}%~%b%F{reset}%(#.#.$) '
            RPROMPT=
            ;;
        backtrack)
            PROMPT=$'${debian_chroot:+($debian_chroot)}${VIRTUAL_ENV:+($(basename $VIRTUAL_ENV))}%B%F{red}%n@%m%b%F{reset}:%B%F{blue}%~%b%F{reset}%(#.#.$) '
            RPROMPT=
            ;;
    esac
    unset prompt_symbol
}

# The following block is surrounded by two delimiters.
# These delimiters must not be modified. Thanks.
# START KALI CONFIG VARIABLES
PROMPT_ALTERNATIVE=twoline
NEWLINE_BEFORE_PROMPT=yes
# STOP KALI CONFIG VARIABLES

if [ "$color_prompt" = yes ]; then
    # override default virtualenv indicator in prompt
    VIRTUAL_ENV_DISABLE_PROMPT=1

    configure_prompt

    # enable syntax-highlighting
    if [ -f /usr/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh ]; then
        . /usr/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh
        ZSH_HIGHLIGHT_HIGHLIGHTERS=(main brackets pattern)
        ZSH_HIGHLIGHT_STYLES[default]=none
        ZSH_HIGHLIGHT_STYLES[unknown-token]=underline
        ZSH_HIGHLIGHT_STYLES[reserved-word]=fg=cyan,bold
        ZSH_HIGHLIGHT_STYLES[suffix-alias]=fg=green,underline
        ZSH_HIGHLIGHT_STYLES[global-alias]=fg=green,bold
        ZSH_HIGHLIGHT_STYLES[precommand]=fg=green,underline
        ZSH_HIGHLIGHT_STYLES[commandseparator]=fg=blue,bold
        ZSH_HIGHLIGHT_STYLES[autodirectory]=fg=green,underline
        ZSH_HIGHLIGHT_STYLES[path]=bold
        ZSH_HIGHLIGHT_STYLES[path_pathseparator]=
        ZSH_HIGHLIGHT_STYLES[path_prefix_pathseparator]=
        ZSH_HIGHLIGHT_STYLES[globbing]=fg=blue,bold
        ZSH_HIGHLIGHT_STYLES[history-expansion]=fg=blue,bold
        ZSH_HIGHLIGHT_STYLES[command-substitution]=none
        ZSH_HIGHLIGHT_STYLES[command-substitution-delimiter]=fg=magenta,bold
        ZSH_HIGHLIGHT_STYLES[process-substitution]=none
        ZSH_HIGHLIGHT_STYLES[process-substitution-delimiter]=fg=magenta,bold
        ZSH_HIGHLIGHT_STYLES[single-hyphen-option]=fg=green
        ZSH_HIGHLIGHT_STYLES[double-hyphen-option]=fg=green
        ZSH_HIGHLIGHT_STYLES[back-quoted-argument]=none
        ZSH_HIGHLIGHT_STYLES[back-quoted-argument-delimiter]=fg=blue,bold
        ZSH_HIGHLIGHT_STYLES[single-quoted-argument]=fg=yellow
        ZSH_HIGHLIGHT_STYLES[double-quoted-argument]=fg=yellow
        ZSH_HIGHLIGHT_STYLES[dollar-quoted-argument]=fg=yellow
        ZSH_HIGHLIGHT_STYLES[rc-quote]=fg=magenta
        ZSH_HIGHLIGHT_STYLES[dollar-double-quoted-argument]=fg=magenta,bold
        ZSH_HIGHLIGHT_STYLES[back-double-quoted-argument]=fg=magenta,bold
        ZSH_HIGHLIGHT_STYLES[back-dollar-quoted-argument]=fg=magenta,bold
        ZSH_HIGHLIGHT_STYLES[assign]=none
        ZSH_HIGHLIGHT_STYLES[redirection]=fg=blue,bold
        ZSH_HIGHLIGHT_STYLES[comment]=fg=black,bold
        ZSH_HIGHLIGHT_STYLES[named-fd]=none
        ZSH_HIGHLIGHT_STYLES[numeric-fd]=none
        ZSH_HIGHLIGHT_STYLES[arg0]=fg=cyan
        ZSH_HIGHLIGHT_STYLES[bracket-error]=fg=red,bold
        ZSH_HIGHLIGHT_STYLES[bracket-level-1]=fg=blue,bold
        ZSH_HIGHLIGHT_STYLES[bracket-level-2]=fg=green,bold
        ZSH_HIGHLIGHT_STYLES[bracket-level-3]=fg=magenta,bold
        ZSH_HIGHLIGHT_STYLES[bracket-level-4]=fg=yellow,bold
        ZSH_HIGHLIGHT_STYLES[bracket-level-5]=fg=cyan,bold
        ZSH_HIGHLIGHT_STYLES[cursor-matchingbracket]=standout
    fi
else
    PROMPT='${debian_chroot:+($debian_chroot)}%n@%m:%~%(#.#.$) '
fi
unset color_prompt force_color_prompt

toggle_oneline_prompt(){
    if [ "$PROMPT_ALTERNATIVE" = oneline ]; then
        PROMPT_ALTERNATIVE=twoline
    else
        PROMPT_ALTERNATIVE=oneline
    fi
    configure_prompt
    zle reset-prompt
}
zle -N toggle_oneline_prompt
bindkey ^P toggle_oneline_prompt

# If this is an xterm set the title to user@host:dir
case "$TERM" in
xterm*|rxvt*|Eterm|aterm|kterm|gnome*|alacritty)
    TERM_TITLE=$'\e]0;${debian_chroot:+($debian_chroot)}${VIRTUAL_ENV:+($(basename $VIRTUAL_ENV))}%n@%m: %~\a'
    ;;
*)
    ;;
esac

precmd() {
    # Print the previously configured title
    print -Pnr -- "$TERM_TITLE"

    # Print a new line before the prompt, but only if it is not the first line
    if [ "$NEWLINE_BEFORE_PROMPT" = yes ]; then
        if [ -z "$_NEW_LINE_BEFORE_PROMPT" ]; then
            _NEW_LINE_BEFORE_PROMPT=1
        else
            print ""
        fi
    fi
}

# enable color support of ls, less and man, and also add handy aliases
if [ -x /usr/bin/dircolors ]; then
    test -r ~/.dircolors && eval "$(dircolors -b ~/.dircolors)" || eval "$(dircolors -b)"
    export LS_COLORS="$LS_COLORS:ow=30;44:" # fix ls color for folders with 777 permissions

    alias ls='ls --color=auto'
    #alias dir='dir --color=auto'
    #alias vdir='vdir --color=auto'

    alias grep='grep --color=auto'
    alias fgrep='fgrep --color=auto'
    alias egrep='egrep --color=auto'
    alias diff='diff --color=auto'
    alias ip='ip --color=auto'

    export LESS_TERMCAP_mb=$'\E[1;31m'     # begin blink
    export LESS_TERMCAP_md=$'\E[1;36m'     # begin bold
    export LESS_TERMCAP_me=$'\E[0m'        # reset bold/blink
    export LESS_TERMCAP_so=$'\E[01;33m'    # begin reverse video
    export LESS_TERMCAP_se=$'\E[0m'        # reset reverse video
    export LESS_TERMCAP_us=$'\E[1;32m'     # begin underline
    export LESS_TERMCAP_ue=$'\E[0m'        # reset underline

    # Take advantage of $LS_COLORS for completion as well
    zstyle ':completion:*' list-colors "${(s.:.)LS_COLORS}"
    zstyle ':completion:*:*:kill:*:processes' list-colors '=(#b) #([0-9]#)*=0=01;31'
fi

# some more ls aliases
alias ll='ls -l'
alias la='ls -A'
alias l='ls -CF'

# enable auto-suggestions based on the history
if [ -f /usr/share/zsh-autosuggestions/zsh-autosuggestions.zsh ]; then
    . /usr/share/zsh-autosuggestions/zsh-autosuggestions.zsh
    # change suggestion color
    ZSH_AUTOSUGGEST_HIGHLIGHT_STYLE='fg=#999'
fi

# enable command-not-found if installed
if [ -f /etc/zsh_command_not_found ]; then
    . /etc/zsh_command_not_found
fi

[[ -s "/etc/grc.zsh" ]] && source /etc/grc.zsh
```

- Apply the changes:

```bash
source ~/.zshrc
```

- Reboot to apply shell change:

```bash
reboot
```

> [!NOTE]
> **📸 Screenshot**
> _Capture: Kali Zsh two-line prompt showing the coloured brackets, user@host segment, working directory, and syntax-highlighted command line_

## Configuration — Revert Back to Bash

- Find the Bash binary:

```bash
which bash
```

- Change the default shell back to Bash:

```bash
chsh -s $(which bash)
```

- Verify the change:

```bash
echo $SHELL
```

## Best Practices

- **Back up `~/.zshrc` before large edits.** The Kali reference config is interdependent (prompt functions, keybindings, completion) — keep a copy so a bad edit is easy to revert.
- **Prefer `setopt hist_ignore_space`** (already enabled above) so commands you prefix with a space stay out of history — useful when a command line contains a secret.
- **Test with `exec zsh` before `chsh`.** Confirm the config loads cleanly in a subshell so a syntax error in `~/.zshrc` cannot break your next login.

## Security Considerations

- **`curl | sh` installers are risky.** The Oh My Zsh install pipes a remote script straight into a shell. Review the script first on hardened hosts, and pin to a known revision where possible.
- **History leaks credentials.** `~/.zsh_history` records commands verbatim. Rely on `hist_ignore_space`, avoid inline passwords/tokens, and restrict permissions (`chmod 600 ~/.zsh_history`).
- **Anything sourced from `~/.zshrc` runs at every interactive login** — including `/etc/grc.zsh` and the syntax-highlighting/autosuggestion plugins. Treat these files as a persistence surface and verify their integrity during hardening reviews.
- **Do not set Zsh as the shell for non-interactive service accounts;** use `/usr/sbin/nologin` for accounts that should never obtain a shell.

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| `chsh: shell not found in /etc/shells` | Zsh path missing from `/etc/shells` | `echo $(which zsh) >> /etc/shells`, retry |
| Default shell unchanged after `chsh` | Change applies at next login | Log out/in, or `exec zsh` |
| Prompt broken / plain after editing `.zshrc` | Syntax error or missing plugin file | Compare against the reference; check the `.zsh` plugin paths exist |
| No syntax highlighting / autosuggestions | Plugin packages not installed | Install `zsh-syntax-highlighting` and `zsh-autosuggestions` |
| grc colors absent | `/etc/grc.zsh` missing | Confirm grc is installed and the file exists — see [Enhance-Bash-with-grc](Enhance-Bash-with-grc.md) |

## References

- `man 1 zsh`, `man 1 zshoptions`, `man 1 zshzle`
- Oh My Zsh: https://github.com/ohmyzsh/ohmyzsh
- Kali default `.zshrc`: `/usr/share/doc/zsh/examples/zshrc`

## Related

- [Shells-in-Linux](Shells-in-Linux.md) — shell overview and configuration.
- [Fish-Shell](Fish-Shell.md) — comparable modern interactive shell.
- [Linux-Environment-Variables](Linux-Environment-Variables.md) — configure the shell environment.
- [Enhance-Bash-with-grc](Enhance-Bash-with-grc.md) — colorize command output (`/etc/grc.zsh`).
- [Linux Administration & Server Hardening](../Readme.md) — Linux administration & server hardening hub.
