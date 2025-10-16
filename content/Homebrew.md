---
{"publish":true,"aliases":"","created":"2025-06-29 22:42","modified":"2025-10-16T14:46:54.370+02:00","cssclasses":""}
---


## Install

### [[Fedora]]

```sh
sudo dnf group install 'Development Tools'
sudo dnf install procps-ng curl file
```

```sh
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

## Configure

### [[fish]]

Normally you would have to run

```fish
eval $(/home/linuxbrew/.linuxbrew/bin/brew shellenv)
```

each time, as the `shellenv` command from brew doesn't populate _universal_ variables.

If you use my [[Dotfiles]] however, it's already automagically evaluated as soon as the `/home/linuxbrew/.linuxbrew/bin/brew` executable is found.
