---
{"publish":true,"created":"2025-05-30 16:17","cssclasses":""}
---


# Install

## #OS/Fedora 

```sh
sudo dnf copr enable pgdev/ghostty
sudo dnf install ghostty
```

I also need to install my currently used font package [[Iosevka\|Iosevka]] inside WSL, otherwise Ghostty will fall back to its included [[JetBrains Mono\|JetBrains Mono]] font.