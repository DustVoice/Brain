---
publish: true
aliases: ""
created: 2025-06-10 14:04
modified: 2025-10-16T14:46:54.460+02:00
cssclasses: ""
---


A Window Manager for both X11 and Wayland, written in Python.

## Install

### [[Fedora]]

```sh
sudo dnf install qtile python-pywlroots
```

## Start

### Wayland

To start Qtile in Wayland mode (the only one working under [[Fedora WSL]] for me), simply use the `-b` flag

```sh
qtile start -b wayland
```
