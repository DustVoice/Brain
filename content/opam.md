---
{"publish":true,"aliases":"","created":"2025-08-06 13:52","modified":"2025-10-16T14:46:54.191+02:00","cssclasses":""}
---


## Install

### [[Fedora]]

```sh
sudo dnf install ocaml ocaml-findlib opam
```

### Generic Linux

Normally it is recommended to install opam using the following shell script snippet, to get the latest version, etc.

```sh
bash -c "sh <(curl -fsSL https://opam.ocaml.org/install.sh)"
```

You need to have a few dependencies installed though, so make sure to install them as well:

> [!todo] Install
> - [[gcc]]
> - [[build-essential]]
> - [[curl]]
> - [[unzip]]
> - [[bubblewrap]]

## Initialize

> [!NOTE]
> If you use my [[Dotfiles]], together with [[fish]], I already include the relevant `eval` statement the following command wants to add to the config file.
> You can therefore simply skip that step.

```sh
opam init
```
