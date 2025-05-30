---
{"publish":true,"created":"2025-05-30 17:21","cssclasses":""}
---


# Install

## #OS/Fedora 

> [!note]
> Should already be installed and up-to-date

```sh
sudo dnf install gnupg
```

# Setup

Initialize a keyring by issuing a command that expects it

> [!example]
> ```sh
> gpg -K
> ```

## Smart Card

### Get it working

To set up a smart card, first make sure it appears in `lsusb`.

Then make sure it also shows in `gpg --card-status{:sh}`.
This command _should_ output some information about the present smart card. If the output looks like the desert scene out of an old western movie, it’s probably not being recognized correctly.

To use the smart card, we need to make sure [[pcsc\|pcsc]] which manages the smart card, as well as [[ccid\|ccid]], which is a generic CCID driver, are available, configured, [[pcsc#Enable\|enabled]] and [[pcsc#Start\|started]].

### Register

The final thing to do now, is to register the smart card and the key to `gpg`.

The easiest way for me, since I have set the _URL of the public key_ attribute on my YubiKey, is to

1. `gpg --card-edit{:sh}`
2. `fetch` the public key
3. `quit` out of the prompt
4. `gpg --edit-key email{:sh}`, replacing `email` with your actual email
5. `trust` the key with ultimate (`5`) trust and confirming (`y`) it
6. `save` the changes
7. Confirm everything worked by using `gpg -K`

### Pinentry

You might want to install a [[Pinentry\|GUI pinentry]].