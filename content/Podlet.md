---
{"publish":true,"aliases":"","created":"2025-05-09 10:35","modified":"2025-10-16T14:46:54.170+02:00","cssclasses":""}
---


To quickly and easily generate [Podman Quadlet](https://www.redhat.com/en/blog/quadlet-podman) files using a Podman command, you can use [podlet](https://github.com/containers/podlet).

> [!note] Deprecation Notice
> I have migrated away from using podlet, as I found it too much of a hassle.
> I often have to edit the generated file anyway and with some options being straight up unavailable (yet?), I found it much easier, to quickly whip up the Quadlet file manually.
>
> It is however still a valuable tool I use, if I encounter a tutorial or documentation which specifies the docker or podman commands, with all arguments, themselves.
> In that case, you simply prepend it with `podlet` and you're almost good to go.

## Install

### [[Fedora]]

```sh
sudo dnf install podlet
```
