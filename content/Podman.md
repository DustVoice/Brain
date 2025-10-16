---
{"publish":true,"aliases":"","created":"2025-05-02 10:46","modified":"2025-10-16T14:46:54.290+02:00","cssclasses":""}
---


## Install

### [[Fedora]]

```sh
sudo dnf install podman
```

## Rootless

I like to use podman rootless, to further contain and separate containers.

### Reload the daemon

As Quadlet files are `systemd` service files, you need to reload the daemon.

```sh
systemctl --user daemon-reload
```

This generates appropriate `.service` files.

> [!tip]
> Sometimes, this can fail and _not_ generate a `.service` file.
> To debug this, immediately drop into the user journal, to see any error messages
>
> ```sh /;/
> systemctl --user daemon-reload --no-block; journalctl --user -f
> ```

### Enable the service

```sh
systemctl --user enable name.service
```

### Start the service

```sh /name/
systemctl --user start name.service
```

### Check the status

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

> [!tip]
> Sometimes, the non-service-specific journal can be helpful in debugging a problem.
> In that case, simply restart the service and immediately drop into the journal:
>
> ```sh /;/
> systemctl --user restart name.service --no-block; journalctl --user -f
> ```

### Keep it running

As a rootless setup doesn't use a system-level service, all services would be stopped upon logout.

To prevent this, we must `enable-linger` (where `user` is your username, of course):

```sh /user/
loginctl enable-linger user
```

### Auto-Update

If you enabled the auto-update feature using the `AutoUpdate` key in the `.container` file, you still need to enable the auto-update timer

```sh
systemctl --user enable --now podman-auto-update.timer
```
