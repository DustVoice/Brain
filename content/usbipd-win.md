---
{"publish":true,"created":"2025-05-30 16:36","cssclasses":""}
---


USB/IP basically forwards USB over IP/Network, as the name implies.
Forwarding or sharing sockets, or similar, is an absolute nightmare, so this enables a quick and easy operation.

The only downside is that the device won’t be available on Windows anymore.
This is either the case _while the device is attached_, or even _as long as it’s bound_ (see below for further explanation). This I can live with, as my whole development environment lives within WSL anyway.

# Install

> [!attention]
> You **need** to perform the [[usbipd-win#Windows\|#Windows]] installation in addition to the distro's specific instructions.

## Windows

First, you need to install the [usbipd-win](https://github.com/dorssel/usbipd-win) software on the Windows side.
Some [[usbipd-win#GUI\|#GUI]] tools automatically install this.

You can also run (from an **elevated** prompt)

```ps
winget install usbipd
```

> [!tip]
> Make sure to reopen any shell/terminal and check that `uspipd.exe` is available on your `PATH`.


# Set up

> [!note]
> You might need to perform some or even all of the following steps using **elevated** rights.

## GUI

The process can be greatly simplified, by installing downloading [wsl-usb-manager](https://github.com/nickbeth/wsl-usb-manager) on the Windows side.

> [!note]
> This GUI does not automatically install `usbipd-win`!
> Please download the installer from the [release page](https://github.com/dorssel/usbipd-win/releases) and install it.

You can do everything you desire from within the GUI, even set up auto attachments, without a hassle.

## CLI

First, list available devices using

```ps
usbipd.exe list
```

Take note of the `BUSID` of the desired (smart card) device.

> [!tip]
> You can also use the _Hardware ID_ (`VID:PID` column) by using the `-i` flag instead of the `-b` one.

Then simply execute

```ps /BUSID/
usbipd.exe bind -b BUSID
```

> [!todo] Replace
> - `BUSID` : the one you took note of earlier.

> [!note]
> It might be necessary to _force_ this operation with the `-f` flag, in which case the device will be unavailable under Windows until you perform an `unbind`.

Now the only thing left to do is attaching the smart card to WSL

```ps
usbipd.exe attach -w -b BUSID
```

You can even have it auto-attached with

```ps
usbipd.exe attach -w -a -b BUSID
```

> [!note]
> The `bind` procedure is persistent between reboots. The `attach` procedure however isn’t, necessitating a reattachement.
> 
> This also means forcing the `bind` using `-f` makes it unavailable _until_ you explicitly do an `unbind`, when `attach` on the contrary, reinstates control to Windows whenever the device is no longer connected to WSL.

> [!tip]
> You can easily check if this worked by observing the `usbipd.exe` command output, or by utilizing `lsusb` from the [`usbutils`](https://archlinux.org/packages/core/x86_64/usbutils/) package.

# FIDO2

Despite all this, some utilities, for example `ssh-keygen`, utilize a direct smart card access using, e.g., the `/dev/hidraw` devices the USB/IP setup produces.

For that to work, I had to set up a [[Yubico YubiKey#`udev` rule|udev rule]].