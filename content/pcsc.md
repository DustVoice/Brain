---
{"publish":true,"aliases":"","created":"2025-05-30 17:30","modified":"2025-09-15T14:55:13.764+02:00","cssclasses":""}
---


## Install

### #OS/Fedora

> [!note]
> Should already be present

```sh
sudo dnf install pcsc-lite
```

## Configure

### Polkit

You need to add a polkit rule `/etc/polkit-1/rules.d/99-pcscd.rules`, to allow the users of the `smartcard` group access to the smart card

```js title="/etc/polkit-1/rules.d/99-pcscd.rules"
polkit.addRule(function (action, subject) {
  if (
    action.id == "org.debian.pcsc-lite.access_card" &&
    subject.isInGroup("smartcard")
  ) {
    return polkit.Result.YES;
  }
});
polkit.addRule(function (action, subject) {
  if (
    action.id == "org.debian.pcsc-lite.access_pcsc" &&
    subject.isInGroup("smartcard")
  ) {
    return polkit.Result.YES;
  }
});
```

Restart the polkit service

```sh
sudo systemctl restart polkit.service
```

Create the group

```sh
sudo groupadd smartcard
```

Add your user to the group

```sh /username/
sudo usermod -aG smartcard username
```

> [!todo] Replace
> - `username` : Your username

Restart your login session

## Enable

```sh
sudo systemctl enable pcscd
```

## Start

```sh
sudo systemctl start pcscd
```
