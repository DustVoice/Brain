---
{"publish":true,"aliases":"","created":"2025-05-30 14:00","modified":"2025-09-19T10:52:21.896+02:00","cssclasses":""}
---


> [!question] What are  p dotfiles?
> The files used to configure settings of applications, which are often the main _personalization_ component of a Linux installation, are often referred to as _dotfiles_.
>
> This is because most of these files' filename (or the folders they are contained in) starts with a dot.
> Under a UNIX operating system, this usually denotes a _hidden_ file, not visible in file explorers or similar _by default_.
> This is in line with other operating systems which also hide configuration files from the average user (e.g., Windows and it's `%AppData%` folder).

To manage my personal dotfile collection, I currently employ the help of [[Chezmoi]].

My dotfile collection grew so much, that I transitioned from a manual approach to an automated deployment and management with this tool.

Make sure to [[Chezmoi#Install and Apply]] my (or you) dotfile repo to proceed.

> [!note] Reapplying Chezmoi
> You might have to re[[Chezmoi#Apply\|apply]].
> This is particularly true, after installing, for example, the [[fish#Additional Tools]].
>
> This is because there are template sections within my chezmoi files, which check for the availability of a specific command (e.g., `oh-my-posh` / [[oh-my-posh]]), before inserting it into the _real_ file.

## Neovim

I normally use a [[Neovim]] config that makes use of a couple of additional tools, so it might make sense to ensure that they're installed and set up.

> [!todo] Install and Configure
> - [[wget]]
> - [[ripgrep]]
> - [[fd]]
> - [[fzf]]
> - [[xz]]
> - [[Jujutsu]]
> - [[NodeJS]]
> - [[Python]]
> - [[Tree-sitter]]
> - _[[Lua]]_
> - _[[lazyjj]]_
