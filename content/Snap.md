---
{"publish":true,"aliases":"","created":"2025-08-01 08:56","modified":"2025-09-19T10:52:21.325+02:00","cssclasses":""}
---


Snap is a package manager developed by Canonical, read #OS/Ubuntu.
It aims to provide an easy installation of self-contained packages across Linux distributions and systems, similar to [[Flatpak]].

## Install

### #OS/Fedora

```sh
sudo dnf install snap
```

## Configure

### Classic

If you need to install snaps with the `--classic` option, you need to make `/snap` available.
Simply symlink it from `/var/lib/snapd/snap`:

```sh
sudo ln -s /var/lib/snapd/snap /snap
```

Make sure you also add `/snap/bin` to your `$PATH`.

### Proxy

If you're behind a proxy, you can use

```sh
sudo snap set system proxy.http="$HTTP_PROXY"
sudo snap set system proxy.https="$HTTPS_PROXY"
```
