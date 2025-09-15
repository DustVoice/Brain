---
{"publish":true,"aliases":"","created":"2025-06-25 06:53","modified":"2025-09-15T14:55:13.868+02:00","cssclasses":""}
---


Fish is a _shell_, more specific, it's the “**f**riendly **i**nteractive **sh**ell”.

It is a _non_-POSIX compliant shell that focuses on user-friendliness, usability, ease of use, easy scriptability of the shell (not necessarily as extensive as [[Nushell]]).
It is particularly known for its superb autocompletion, command suggestions and tab completion.

Not only that, but it also features a unique concept, where you can, for example, set environment variables _globally_, which means they persist longer than the shell session.
In reality, fish simply populates a file with all these settings, which is, however, separate to the fish _config_.
This is particularly useful for me, as I manage my dotfiles using [[Chezmoi]], so any change to the config file is considered a _conflict_ or _change_ by chezmoi.
I, however, don't necessarily need these specific variables to be present on other machines.

In other words, it **Just Works™**, at least when it comes to the features specified above, without the need for an elaborate setup like with [[ZSH]], for example.
And although I ship my custom config with my [[Dotfiles]], requiring [[fish#Additional Tools]], fish is highly usable out of the box, with sane defaults and easy configuration.

## Install

### #OS/Fedora

```sh
sudo dnf install fish
```

### [[Homebrew]]

If the latest `fish` shell isn't available for your distribution or version, you can try installing it using [[Homebrew]]:

```sh
brew install fish
```

## Update Completions

You might also want to update completions (automagically), especially after installing new software.

```fish
fish_update_completions
```

## Additional Tools

My specific [[Dotfiles]], use various tools, so it's best to make sure they're available and set up.

> [!todo] Install
> - [[oh-my-posh]]
> - [[zoxide]]
> - _[[Zellij]]_

## Plugins

There's also a plugin manager for fish, called [[Fisher]].

> [!todo] Setup
> - [[Fisher]]

> [!note]
> After deploying my [[Dotfiles]], as well as installing [[Fisher]], I usually install the following plugins.
>
> - [[fish#Catppuccin]]
> 	- You still have to [[fish#Save it]]!
> - [[fish#Sponge]]
> - [[fish#Pufferfish]]
> - [[fish#Replay]]
> - [[fish#Eza]]
> - [[fish#fzf]]
> - [[fish#jj]]

### Catppuccin

[Catppuccin](https://github.com/catppuccin/fish) is my theme of choice. Everywhere.

#### Install

```fish
fisher install catppuccin/fish
```

#### Save it

```fish
fish_config theme save "Catppuccin Macchiato"
```

### Sponge

To keep my typos out of the shell history, I use [Sponge](https://github.com/meaningful-ooo/sponge).

```fish
fisher install meaningful-ooo/sponge
```

> [!info]- Effect
> Deletes typos from the command history, but always keeps the last 2 commands alive.

### Pufferfish

Some neat text expansion. See the [project's GitHub](https://github.com/nickeb96/puffer-fish) for more information.

```fish
fisher install nickeb96/puffer-fish
```

> [!info]- Substitutions
> | Input | Substitution |
> | -- | -- |
> | `..[.](n times)` | `..[/..](n times)` |
> | `...` | `../..` |
> | `....` | `../../..` |
> | ===== | ===== |
> | `!!` | Previous command |
> | `apt upgrade` | Command |
> | `sudo !!` | `sudo apt upgrade` |
> | ===== | ===== |
> | `!$` | Last argument |
> | `sudo dnf install nushell` | Command attempt |
> | `cargo install !$` | Reuse last argument |

### Replay

[Replay](https://github.com/jorgebucaran/replay.fish) enables you to execute bash snippets and proceed with an up-to-date environment.

```fish
fisher install jorgebucaran/replay.fish
```

> [!info]- Example
>
> ```shell-session
> $ pwd
> /home/users/jb/replay.fish
> $ replay cd ~
> $ pwd
> /home/users/jb
> ```

### Bass

An alternative to [[fish#Replay]] is [bass](https://github.com/edc/bass).

```fish
fisher install edc/bass
```

### Eza

[fish-eza](https://github.com/plttn/fish-eza) is a neat little [[eza]] integration. Make sure the dependencies are set up

> [!todo] Install
> - [[eza]]

```fish
fisher install plttn/fish-eza
```

> [!info]- Base aliases
> |alias|default options|
> |---|---|
> |`l`|`eza`|
> |`ll`|`eza --group --header --group-directories-first --long`|
> |`ll` in git repo|`eza --group --header --group-directories-first --long --git`|
> |`lg`|`eza --group --header --group-directories-first --long --git --git-ignore`|
> |`le`|`eza --group --header --group-directories-first --long --extended`|
> |`lt`|`eza --group --header --group-directories-first --tree --level LEVEL`|
> |`lc`|`eza --group --header --group-directories-first --across`|
> |`lo`|`eza --group --header --group-directories-first --oneline`|

> [!info]- Alias suffixes
>
> |Extend suffix|Default options|
> |---|---|
> |`a`|`--all --binary`|
> |`d`|`--only-dirs`|
> |`i`|`--icons`|
> |`id`|`--icons --only-dirs`|
> |`aa`|`--all --binary --all`|
> |`ad`|`--all --binary --only-dirs`|
> |`ai`|`--all --binary --icons`|
> |`aid`|`--all --binary --icons --only-dirs`|
> |`aad`|`--all --binary --all --only-dirs`|
> |`aai`|`--all --binary --all --icons`|
> |`aaid`|`--all --binary --all --icons --only-dirs`|

### fzf

[fzf.fish](https://github.com/PatrickF1/fzf.fish) is a nice [[fzf]] integration. Make sure the dependencies are set up beforehand

> [!todo] Install
> - [[fzf]]
> - [[fd]]
> - [[bat]]

```fish
fisher install PatrickF1/fzf.fish
```

> [!info]- Keybinds
> | Keybind    | Effect            |
> | ---------- | ----------------- |
> | Ctrl+Alt+F | Search Directory  |
> | Ctrl+Alt+L | Search Git Log    |
> | Ctrl+Alt+S | Search Git Status |
> | Ctrl+R     | Search History    |
> | Ctrl+Alt+P | Search Processes  |
> | Ctrl+V     | Search Variables  |

### jj

[plugin-jj](https://github.com/kapsmudit/plugin-jj) for some [[Jujutsu]] integration.

```fish
fisher install kapsmudit/plugin-jj
```

> [!info]- Abbreviations
> |Abbreviation|Command|
> |---|---|
> |jst|jj status|
> |jsh|jj show|
> |jshs|jj show --summary|
> |jl|jj log|
> |jbm|jj bookmark move|
> |jj bm|bookmark move|
> |jcl|jj git clone --colocate|
> |jclo|jj git clone --colocate --remote upstream|
> |jd|jj desc|
> |jdm|jj desc -m|
> |jdf|jj diff|
> |jdg|jj diff --git|
> |je|jj edit|
> |jj e|edit|
> |jfa|jj git fetch --all-remotes|
> |jf|jj git fetch|
> |jp|jj git push|
> |jn|jj new|
> |jna|jj new -A|
> |jnb|jj new -B|
> |jnn|jj new --no-edit|
> |jnna|jj new --no-edit -A|
> |jnnb|jj new --no-edit -B|
> |jsq|jj squash|
> |jgr|jj git remote|
> |jgrl|jj git remote list|
> |jrb|jj rebase|
> |ja|jj abandon|

### Abbreviation Tips

[fish-abbreviation-tips](https://github.com/gazorby/fish-abbreviation-tips) is a really useful tool to get into the habit of using _your_ and _your plugin's_ abbreviations.
Because, if you don't use them, you might as well get rid of them.

```fish
fisher install gazorby/fish-abbreviation-tips
```

## Login Shell

If you follow a [[System Administration]] [[Guide]], I normally set the login shell to `bash` when creating the custom user, so you might wonder why I don’t directly set it to `fish`.

Well, fish **isn’t POSIX-compliant**, and neither does it want to be. Therefore, running `fish` as a login shell might not be the absolute best experience you’ll ever have.

Instead, I include a code snippet at the bottom of my `~/.bashrc`, below the interactive check `[[ $- == *i* ]]`, which will let `fish` take over any _interactive_ shell, while scripts, etc. that expect a `POSIX` compliant shell can have their way.

```bash title="~/.bashrc" {3-8}
[[ $- == *i* ]] || return

if [[ $(ps --no-header --pid=$PPID --format=comm) != "fish" && -z ${BASH_EXECUTION_STRING} && ${SHLVL} == 1 ]]
then
        shopt -q login_shell && LOGIN_OPTION='--login' || LOGIN_OPTION=''
        FISH_BIN=$(command -v fish)
        [ -x "$FISH_BIN" ] && SHELL=${FISH_BIN} exec $FISH_BIN "$LOGIN_OPTION"
fi
```
