---
{"publish":true,"created":"2025-05-30 12:01","cssclasses":""}
---

# About

Guides are a pretty linear subset of my notes.

I try to abstract and extract as much information as possible into self-defined notes, although that's not always possible (right away).

They are mainly intended for, e.g., preserving information about my setups, specific information about my architecture, or useful information, if I ever need to change or reinstall anything.
Therefore, they are mainly intended as a set of personal mental crutches and references, so the following disclaimer applies:

> [!danger]
> - Your mileage may vary!
> - Not for the faint of heart!
> - Be wary of utter steaming hot garbage!
> - Proceed with caution and at your own risk!

# Syntax

Throughout my guide, I use some special elements to make it easier to retrieve certain information at a glance.

## Prerequisites

At the beginning of a (sub-)section, there might be an **Install** or **Configure** todo section present.

This is an accumulated list of packages/tools mentioned throughout a chapter.
The packages installed/configured in these notes are often required/assumed to be present from this point forward, or are configured, etc.
It makes it easier to see which packages/tools are supposed to be installed at a glance, so nothing is forgotten.

> [!NOTE]
> Note that many of these package notes have sections for different operating systems or distributions.
> Normally, the section for the distribution of the guide, as well as the intended operation (install, configure, …) should be linked.

Optional packages are _italicized_.

> [!todo] Install
> - [[Podman#OS/Fedora\|Podman#OS/Fedora]]
> - _[[Caddy\|Caddy]]_

## Callouts

I use various callouts.
Most of them are pretty self-explanatory.
Nevertheless, I try to stick to a general set of rules for them:

> [!info]
> General useful information or detailed explanation, that isn't immensely important when skimming or reading again.

> [!NOTE]
> Highlights information that users should take into account, even when skimming.

> [!abstract]
> A **TL;DR**, if I feel a note has evolved into a behemoth or got blown out of proportion.

> [!todo]
> Things to install, configure, change, etc., before proceeding, or before executing a command.

> [!example]
> An example, not intended to be used directly or simply copy/pasted, rather to be used as a template or pattern.

> [!tip]
> Optional information to help a user be more successful.

> [!warning]
> Critical content demanding immediate user attention due to potential risks.

> [!danger]
> Negative potential consequences of an action.

## Terminal and Code Blocks

I try to enhance the code blocks with some useful information, such as filename, highlighted changes, etc.

Executing a command within Windows, for example, will normally be marked by a block, highlighted with `PowerShell`-specific syntax:

```ps
Get-Language
which.exe wsl
```

Executing a command from within WSL will be highlighted as `bash`-syntax (or `nu`shell-syntax).

```sh
ls -la | grep ".el"
```

If a file or code snippet is written in a specific language, it will also be highlighted appropriately

```rust
println!("Hello world");
```

A file path can optionally be specified:

```text title="~/log.txt"
[INFO:] Test
```