---
{"publish":true,"created":"2025-05-30 14:45","cssclasses":""}
---


For easy file editing, I’m using [neovim](https://neovim.io/), so editing is as simple as calling `nvim` on the file.

# Install

## #OS/Fedora 

```sh
sudo dnf install neovim
```

# WSL Clipboard integration

For Neovim’s clipboard integration, you have to decide whether you want native clipboard integration with Wayland/X11, or not.

As a rough measure, you probably want clipboard integration over Windows if you use, e.g., WezTerm installed on Windows to access WSL and you most likely want native clipboard integration for Wayland/X11 if you use an installation of a terminal emulator from out of WSL.

> [!note]
> This doesn’t really seem to be the case anymore, as I currently use [[Ghostty]] from within WSL, having no “native” solution installed, but rather [[win32yank]] on the Windows side, with it on my WSL `PATH` and copy/pasting works like a blast.

## Clipboard via Windows

Set up [[win32yank]] and make sure that your Windows `PATH` (or specifically win32yank's path) is available from within WSL.

## Native clipboard

Set up [[Clipboard Handling]].

# [[Copilot]]

In my [[Dotfiles]], specifically in my Neovim configuration, I configure [[Copilot]].

If you have a GitHub account, Copilot should be available to you.

To authenticate with Copilot, you simply have to

1. Run `:Copilot auth` within Neovim.
2. Copy the displayed one-time-code.
3. Go to https://github.com/login/device.
4. Log into your GitHub account.
5. Paste the previously copied one-time-code.
6. Return to your Neovim instance.
7. Wait until the popup closes.
