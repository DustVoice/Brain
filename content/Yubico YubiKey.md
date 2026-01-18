---
publish: true
aliases: YubiKey
created: 2025-05-30 16:33
modified: 2025-09-19T10:52:21.103+02:00
cssclasses: ""
---


## SSH key

> [!note] WSL
> If you're on WSL, make sure, that `ssh-keygen` can access your device.
> On my WSL setups, I had to create a [[Yubico YubiKey#`udev` rule|udev rule]].

### Generating a key

We use `ssh-keygen` for that.

```sh /username/ /hostname/ /YubiKey description/
ssh-keygen -t ed25519-sk -O resident -O verify-required -C "username@hostname (Yubikey description)"
```

> [!info]- Detailed command dissection
> - Use _Ed25519_ as the algorithm, the `-sk` suffix tells `ssh-keygen` that it is a **s**ecurity **k**ey backed key.
> 	- `-t ed25519-sk`
> - Require PIN entry **and** touch confirmation.
> 	- `-O verify-required`
> - _Optional:_ Create a resident key, discoverable according to your YubiKey's setting
> 	- `-O resident`
> - _Optional:_ Specify the username, hostname, and description of the YubiKey (e.g., _Work YubiKey 5C NFC_) in a comment, to identify the generated key in a `~/.ssh/authorized_keys` file, for example.
> 	- `-C "username@hostname (Yubikey description)"`

> [!note]
> Please also refer to [Yubico's official documentation](https://developers.yubico.com/SSH/Securing_SSH_with_FIDO2.html).

Simply choose a path for the key to be saved to and _optionally a password_. As the file is just a key stub, though, you shouldn't need one if you specified `verify-required`. It _should_ be sufficiently protected, provided you have a sufficiently secure PIN and physical access control to your YubiKey.

> [!warning] Resident Keys
> For my [[Fedora Server#Secure the SSH\|server]], I don't use resident keys.
> Although it might not really be a security concern, I don't have a problem manually deploying the key stub to any PC I want to access my server from.

Depending on whether you chose the default name or not, you might have to add the key to the ssh-agent

```sh /PRIVATE_KEY_FILE/
ssh-add ~/.ssh/PRIVATE_KEY_FILE
```

> [!todo] Replace
> - `PRIVATE_KEY_FILE`: The filename you chose previously

### `udev` rule

Add a `udev` rule in the form of a file `/etc/udev/rules.d/99-yubikey.rules`

```text title="/etc/udev/rules.d/99-yubikey.rules"
KERNEL=="hidraw*", SUBSYSTEM=="hidraw", MODE="0660", TAG+="uaccess", GROUP="plugdev", ATTRS{idVendor}=="1050", ATTRS{idProduct}=="0406"
```

> [!note]
> You might need to change the vendor and product ID.
>
> You can easily check the IDs using `lsusb`.
>
> Simply locate the YubiKey line in the output, and locate the IDs following the pattern: `[Bus IDs]: ID <vendor>:<product> [Name of the device]`

Subsequently, add your user to the `plugdev` group, restart WSL and you should be good to go.

```sh /username/
sudo groupadd plugdev
sudo usermod -aG plugdev username
```

> [!todo] Replace
> - `username` : Your username
