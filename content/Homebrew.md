---
{"publish":true,"created":"2025-06-29 22:42","modified":"2025-07-30T14:36:58.873+02:00","cssclasses":""}
---


# Install

## #OS/Fedora 

```sh
sudo dnf group install 'Development Tools'
sudo dnf install procps-ng curl file
```

```sh
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

# Configure

## [[fish]]

```fish
eval $(/home/linuxbrew/.linuxbrew/bin/brew shellenv)
```