---
{"publish":true,"created":"2025-06-10 14:04","cssclasses":""}
---


A Window Manager for both X11 and Wayland, written in Python.

# Install

## #OS/Fedora 

```sh
sudo dnf install qtile python-pywlroots
```

# Start

## Wayland

To start Qtile in Wayland mode (the only one working under [[Fedora WSL]] for me), simply use the `-b` flag

```sh
qtile start -b wayland
```