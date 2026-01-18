---
publish: true
aliases: ""
created: 2025-05-07 22:27
modified: 2025-10-16T14:46:54.772+02:00
cssclasses: ""
---


> [!question] Why as a system package?
> I installed _Fail2Ban_ as a system package together with a system-wide / `root`-level configuration, because I want a malicious attacker to be stopped as soon as possible, instead of further down the chain.
> I use, e.g., [[Firewalld\|firewalld]] on a `root`-level, too for the exact same reason.

## Install

### [[Fedora]]

```sh
sudo dnf install fail2ban
```

## Firewalld

As I use [[Firewalld\|firewalld]], I make sure `fail2ban-firewalld` was installed together with the main package.
The default `banaction` under `/etc/fail2ban/jail.d/00-firewalld.local` didn't work for me, as it is ignored on forwarded ports.

Therefore, I had to create `/etc/fail2ban/jail.local`, with the following content:

```conf title="/etc/fail2ban/jail.local" {2-3}
[DEFAULT]
banaction = firewallcmd-ipset
banaction_allports = firewallcmd-ipset
```

## Enable

```sh
sudo systemctl enable fail2ban
```

## Start

```sh
sudo systemctl start fail2ban
```

## Restart

After any config change, you need to restart Fail2Ban

```sh
sudo systemctl restart fail2ban
```

## Defaults

I set some (further) defaults in `/etc/fail2ban/jail.local` under the `[DEFAULT]` section

```conf title="/etc/fail2ban/jail.local" {2-7}
[DEFAULT]
maxretry = 5
bantime = 600
findtime = 600
bantime.increment = true
bantime.factor = 1
bantime.multipliers = 1 6 18 144 288 2016 4032 12096
```

The `multipliers` increase the `bantime` (initial time: 600 = 10 minutes) each time, namely to
1. 10 min
2. 60 min
3. 3 hours
4. 1 day
5. 2 days
6. 14 days / 2 weeks
7. 28 days (approx. 1 month)
8. 84 days (approx. 3 months)
