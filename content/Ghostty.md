---
{"publish":true,"created":"2025-05-30 16:17","cssclasses":""}
---


# Install

## #OS/Fedora 

Either use [[Terra]] or [[Copr]].
I recently ran into problems with the [[Copr]] version, so I switched to [[Terra]].

### [[Terra]]

Make sure it is [[Terra#Setup]], then

```sh
dnf install ghostty
```

### [[Copr]]

```sh
sudo dnf copr enable pgdev/ghostty
sudo dnf install ghostty
```

I also need to install my currently used font package [[Iosevka]] inside WSL, otherwise Ghostty will fall back to its included [[JetBrains Mono]] font.