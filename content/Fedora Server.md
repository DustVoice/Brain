---
{"publish":true,"aliases":"","created":"2025-04-29","modified":"2025-09-19T10:52:21.857+02:00","cssclasses":""}
---


![[Disclaimer (Tech)]]

## Why Fedora?

I maintained an Arch Linux server for the longest time.
However, while I love Arch and the way it helped my more profound understanding of Linux, I had to inevitably come to the conclusion that a purely rolling release distribution isn't the best choice for a server environment.

So the search for a new operating system began. And although _NixOS_, with its declarative nature, scratched an itch I didn't know I had, I quickly settled on two options to choose from: _OpenSUSE_ or a _Red Hat distribution_.

_OpenSUSE_ was my first choice after longer consideration and research, but I ran into some problems with my hosting provider and the defaults they used for the installation image.

> [!example]- Example: Btrfs
> [Btrfs](https://wiki.archlinux.org/title/Btrfs) was one of my requirements, ensuring quick and easy rollbacks, paired with all the other benefits it offers.
>
> As I use Btrfs almost exclusively on all my machines nowadays, including Arch Linux, this wasn't a requirement I was willing to waive.
>
> Unfortunately, I wasn't able to set up _OpenSUSE_ on Btrfs with my particular hosting provider.
> I might be or stupid (or both), but I wasn't willing to sink any more time into a resolution.
>
> So I tried _Fedora_: Et voilà, it was quickly and easily installed on Btrfs by default.

Now, a legit question to ask at this point would be:

> Why Fedora and not `<insert another Red Hat distribution here>`?

To be honest, I can't really give an answer. Some years ago, I would've simply installed CentOS and called it a day. But inevitably some time has passed since I last caught up and apparently _Rocky Linux_ is more akin to _CentOS_ nowadays, while _CentOS_ became _CentOS Stream_, sitting more between _Fedora_ and _RHEL_, blah blah blah. In short, it was confusing and a mess at first.

My wish to transition away from a rolling release distribution would've been perfectly met by _Rocky Linux_. **But**, I wanted to consider running a RedHat distro as my main distribution for a while, maybe even on Arch, possibly replacing [[Arch Linux WSL\|my long-time WSL setup using Arch Linux]]. For a day-to-day distro, I definitely wanted to be closer to the _bleeding edge_, even if I were to get some blood on my hands. As Fedora seemed to be fine for a server reinstall, too, I chose it to test the waters.

## Setup Philosophy

Basically, I wanted to try using a containerized setup and installation of all my services, as much as possible. To my shame, I have to admit that I stayed away from using Docker and similar tools as much as I could. It did undoubtedly stem from a lack of understanding and practice, though, so I decided to face my demons, embrace the struggle, enjoy the benefits and have another tool in my toolbox in the end.

This is why big parts of my setup utilize a rootless [podman](https://docs.fedoraproject.org/en-US/fedora-server/containerization/#_podman) setup.

I also decided to try something new, compared to nginx and Apache, so I planned to use [caddy](https://caddyserver.com).

> [!NOTE]
> Note that e.g., Nextcloud, can also be easily installed natively, as outlined by the [Fedora wiki](https://fedoraproject.org/wiki/Nextcloud), which requires a [LAMP stack](https://www.fosslinux.com/93845/how-to-install-lamp-stack-on-fedora.htm), however. As I ran into quite some issues trying to juggle different version requirements for databases, PHP, and more on my old server setup, I wanted to try a containerized approach, (hopefully) mitigating all these issues.

## How it's done

### Create a non-privileged user

Of course, to utilize a _rootless_ podman setup, I first needed to create a new user. Using `root` is generally not recommended anyway.

### Secure the SSH

To further secure my server, I then [[SSH - Key exclusive access\|set up my SSH access]] in a way, where only a login using a [[Yubico YubiKey#SSH key\|SSH key]] is possible.

SSH keys are magnitudes more secure ([it's not even close](https://weberblog.net/passwords-vs-private-keys/), additionally the YubiKey secured ones use _Ed25519_ instead of _RSA_ by default) and the YubiKey brings a second factor to the table.

It doesn't matter to me, whether a bad actor gets access to my “private key” in my `~/.ssh` directory, it's only a _key stub_ anyway.
Even if you'd get access to the key stub, or my PC, you would need
- my YubiKey to be plugged in
- my secure YubiKey PIN to be entered
- physical access to my YubiKey, to press the physical button on it to confirm the operation.

So no way this would work, or at least go unnoticed.

Just for good measure, I also didn't create the keys as _resident keys_.
A resident key, has the key stub **(not the key itself!)** _reside_ on the YubiKey, enabling you to retrieve it on another machine upon entering the correct PIN.
This would in theory allow for the key stub to be retrieved, if I lose the YubiKey and the bad actor gets lucky and guesses the _~80 bit entropy PIN_.

Instead, I need to deploy the key stub from my password vault first, to use the key.

**Even if** the attacker succeeds with all the aforementioned steps, they now only have access to my non-privileged user!
Invoking `sudo` will require them to **also** guess the password of said user (another different one with _~80 bits of entropy_).
And don't even try to get into the `root` account. It's further secured using a YubiKey as a second factor in addition to a password.
And `root` SSH access requires an even more elaborate retrieval of an even more securely stored and protected SSH key, again with no SSH password login possible.

You get the deal.

#### Fail2Ban

You will probably notice, that almost immediately, your `journalctl` will be cluttered with a flood of SSH login attempts.

> [!info]
> This is fully automated and preys on people just setting up a server, choosing an initial weak password, and so on.
> It is not a targeted attack agains you specifically.
> But this level of automation makes it of utmost importance to employ good security measures.

To get rid of this, you can employ [[Fail2Ban]].

After you got it up and running, we simply enable the SSH jail, by creating `/etc/fail2ban/jail.d/sshd.local`:

```conf title="/etc/fail2ban/jail.d/sshd.local"
[sshd]
enabled = true
```

### Rollback

### Podman

![[Podman#Install]]

### Caddy

[[Caddy#Rootless Podman\|Setup Caddy]]

### Nextcloud

[[Nextcloud#Rootless Podman\|Setup Nextcloud]]

### Vaultwarden

> [!missing]- This is currently unused
> For multiple reasons, including but not limited to
> - Unconditional availability
> - Redundancy and frequency of backups
> - Traffic limitations
> I currently don't actively use a self-hosted Vaultwarden instance.
> 
> I trust Bitwarden much more to have reliable backup and uptime solutions employed, together with the manpower of dedicated taskforces.
>
> Cost is also a non-factor, as even with using Vaultwarden, I kept an active subscription with Bitwarden, simply for supporting the effort.
>
> This documentation serves as a knowledge dump, in case I decide to come back to this.

[[Vaultwarden\|Setup Vaultwarden]]
