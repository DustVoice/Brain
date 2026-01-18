---
publish: true
aliases: ""
created: 2025-05-30 16:17
modified: 2025-11-11T13:49:59.061+01:00
cssclasses: ""
---


## Install

### [[Fedora]]

Either use [[Terra]] or [[Copr]].
I recently ran into problems with the [[Copr]] version, so I switched to [[Terra]].

#### [[Terra]]

Make sure it is [[Terra#Setup]], then

```sh
dnf install ghostty
```

#### [[Copr]]

```sh
sudo dnf copr enable pgdev/ghostty
sudo dnf install ghostty
```

I also need to install my currently used font package [[Iosevka]] inside WSL, otherwise Ghostty will fall back to its included [[JetBrains Mono]] font.

## Themes

Themes should come pre-packaged.
However especially under [[Fedora]], there seems to be some packaging issues.
The following script should download the missing themes.

```bash title="ghostty-themes.sh"
#!/bin/bash

mkdir -p ~/.config/ghostty/themes && \
tmpdir=$(mktemp -d) && \
git clone --depth 1 --filter=blob:none --sparse https://github.com/mbadolato/iTerm2-Color-Schemes.git "$tmpdir" && \
cd "$tmpdir" && \
git sparse-checkout set ghostty && \
rsync -a ghostty/ ~/.config/ghostty/themes/ && \
cd ~
rm -rf "$tmpdir"
```
