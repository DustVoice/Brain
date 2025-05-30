---
share: true
created: 2025-05-30 13:50
tags: 
---

> [!info]
> I use a [[Smartcard|Smartcard]] for many recurring and non-recurring tasks in my workflow.
> 
> Therefore, they need to be accessible from within WSL, for me to use WSL as a complete replacement of a Windows dev setup.
> 
> Previously (especially with WSL v1), this was an absolute pain in the butt! With WSL2 and better support overall, it’s actually not that bad.

> [!todo] Install and Configure
> - [[usbipd-win|usbipd-win]]
> - [[GnuPG|GnuPG]]

For [[usbipd-win|usbipd-win]], there should be nothing more to do and should _just work_.

For GPG, most of the configuration is already done by my dotfiles, for example, updating the TTY, which is done in my supplied `~/.bash_profile` file:

```sh
gpg-connect-agent updatestartuptty /bye
```

One thing I do, is to update my [[Chezmoi|Chezmoi]] remote URL, to point to the _SSH Address_ instead of the default _HTTPS_ one, so I can author changes and push them, all using my smart card.

Make sure you have [[SSH|SSH]] installed, as well as [[netcat|netcat]] (the OpenBSD variant), if you're behind a proxy.

```sh /DustVoice/
chezmoi git remote set-url origin git@github.com:DustVoice/dotfiles.git
```

> [!todo] Replace
> - `DustVoice` : With your own GitHub username

Simply [[Chezmoi#Update and refresh externals|Chezmoi > Update and refresh externals]] afterward to make sure everything worked.