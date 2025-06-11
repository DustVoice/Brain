---
{"publish":true,"created":"2025-05-02 10:46","cssclasses":""}
---

# Install

## #OS/Fedora 

```sh
sudo dnf install podman
```

# Rootless

I like to use podman rootless, to further contain and separate containers.

## Reload the daemon

As Quadlet files are `systemd` service files, you need to reload the daemon.

```sh
systemctl --user daemon-reload
```

This generates appropriate `.service` files.

## Enable the service

```sh
systemctl --user enable name.service
```

## Start the service

```sh /name/
systemctl --user start name.service
```

## Check the status

You can check the status of Podman using

```sh
podman ps
```

and the status of the service itself using either

```sh /name/
systemctl --user status name.service
```

or

```sh /name/
journalctl --user -xeu name.service
```

## Keep it running

As a rootless setup doesn't use a system-level service, all services would be stopped upon logout.

To prevent this, we must `enable-linger` (where `user` is your username, of course):

```sh /user/
loginctl enable-linger user
```

## Auto-Update

If you enabled the auto update feature using the `AutoUpdate` key in the `.container` file, you still need to enable the auto update timer

```sh
systemctl --user enable --now podman-auto-update.timer
```