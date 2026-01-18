---
publish: true
aliases: ""
created: 2025-05-30 14:16
modified: 2025-09-19T10:52:22.014+02:00
cssclasses: ""
---


> [!todo] Install
> - [[Git]]

## Install

> [!warning]
> Make sure you are in your `$HOME` directory when running this command!

```sh
sh -c "$(curl -fsLS get.chezmoi.io)"
```

> [!tip]
> You can also [[Chezmoi#Install and Apply]] in a single command!

> [!NOTE]
> Also refer to the [official documentation](https://www.chezmoi.io/install/)

### Install into `~/.local/bin`

```sh "/lb"
sh -c "$(curl -fsLS get.chezmoi.io/lb)"
```

### System Package

#### Arch Linux

## Apply

```sh /DustVoice/
chezmoi init --apply DustVoice
```

> [!todo] Replace
> - `DustVoice` : Your GitHub username, if you maintain a fork, or your own `dotfiles` repository

Quickly and easily applies a config hosted under `https://github.com/DustVoice/dotfiles`.

## Install and Apply

```sh /$install_cmd/ /$apply_cmd/
$install_cmd -- $apply_cmd
```

> [!todo] Replace
> - `$install_cmd` : The appropriate command from the [[Chezmoi#Install]] section
> - `$apply_cmd` : The appropriate command from the [[Chezmoi#Apply]] section, without the `chezmoi`

> [!example]
>
> ```sh
> sh -c "$(curl -fsLS get.chezmoi.io)" -- init --apply DustVoice
> ```

## Update and refresh externals

```sh
chezmoi update -R
```
