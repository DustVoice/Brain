---
share: true
created: 2025-05-02 13:33
tags:
  - fedora
---

![[Disclaimer (Tech)|Disclaimer (Tech)]]

# Rootless Podman

## Prerequisites

Make sure you have [[Podman (Fedora)|podman]] installed and a _frontend_ [[Caddy|Caddy]] instance set up.

## Data directories

First off, create all the necessary directories:

```sh
mkdir -p ~/containers/vaultwarden/{data,env}
```

## Files

Next, we need to create the necessary files.

### Frontend Caddyfile

I use a _frontend_ caddy instance for reverse proxying to Vaultwarden.
Note that the Vaultwarden container expects incoming traffic on port `8000`, as specified in [[#Vaultwarden|its container config]].

Therefore, we simply add a section to the (already present) [[Caddy#Caddyfile|Caddy > Caddyfile]] under `~/containers/caddy/config/Caddyfile`

```text title="~/containers/caddy/config/Caddyfile" /FQDN/
{$VAULTWARDEN_DOMAIN} {
	reverse_proxy http://host.containers.internal:8000 {
		# Send the true remote IP to Rocket, so that Vaultwarden can put this in the
		# log, so that fail2ban can ban the correct IP.
		header_up X-Real-IP {remote_host}
	}
}
```

> [!todo] [[Caddy#Environment variables|Caddy > Environment variables]]
> `VAULTWARDEN_DOMAIN` : [[FQDN|FQDN]] of the Vaultwarden instance

### Environment file

In the [[#Vaultwarden|container file]], we specify an environment file.
This [specifies environment variables available to the container](https://github.com/dani-garcia/vaultwarden/wiki/Enabling-admin-page#secure-the-admin_token).

> [!danger] Secret information
> This file will [contain secret information)(https://github.com/dani-garcia/vaultwarden/wiki/Enabling-admin-page#secure-the-admin_token).
> If you made sure, to [[Server (Fedora)#Secure the SSH|secure your server from outside access]], you should be fine.
> Still, you could consider hardening the access to this file even further.
> You can't however simply only give `root` access to the file, as podman runs _unprivileged_ and won't be able to access the file.
> _SELinux_ might help in that regard.

Create and initially populate the `vaultwarden.env` file under the [[#Data directories|previously created directory]] `~/containers/vaultwarden`

```systemd title="~/containers/vaultwarden/vaultwarden.env"
DOMAIN='https://VAULTWARDEN_DOMAIN'
ROCKET_PORT=8080
LOG_FILE=/var/log/vaultwarden/vaultwarden.log
```

> [!todo] Replace
> `VAULTWARDEN_DOMAIN` : [[FQDN|FQDN]] of this Vaultwarden instance

Vaultwarden will then serve the service over this port _within the container_.
We later redirect an _outside_ port to this in the [[#Vaultwarden|container config]].

## Containers

### Vaultwarden

We simply create a _main Vaultwarden container_.

Create the file under `~/.config/containers/systemd/vaultwarden.container`

```systemd title="~/.config/containers/systemd/vaultwarden.container" /user/
[Unit]
Description=Vaultwarden container
After=network-online.target

[Container]
AutoUpdate=registry
Image=ghcr.io/dani-garcia/vaultwarden:latest
Exec=/start.sh
EnvironmentFile=/home/dustvoice/containers/vaultwarden/vaultwarden.env
Volume=/home/dustvoice/containers/vaultwarden/data:/data:Z
Volume=/home/dustvoice/containers/vaultwarden/logs:/var/log/vaultwarden:z
PublishPort=8000:8080

[Install]
WantedBy=default.target
```

> [!todo] Replace
> - `user` : username used for running the [[Podman (Fedora)#Rootless|rootless podman instance]].
## Boot it up

### Reload

![[Podman (Fedora)#Reload the daemon|Podman (Fedora) > Reload the daemon]]

### Auto-Update

![[Podman (Fedora)#Auto-Update|Podman (Fedora) > Auto-Update]]

### Linger

![[Podman (Fedora)#Keep it running|Podman (Fedora) > Keep it running]]

### Start

![[Podman (Fedora)#Start the service|Podman (Fedora) > Start the service]]

> [!todo] Replace
> `name` : `vaultwarden`

### Status

![[Podman (Fedora)#Check the status|Podman (Fedora) > Check the status]]

> [!todo] Replace
> `name` : `vaultwarden`

### Restart

Following that, you probably still need to restart the _frontend_ [[Caddy|Caddy]], as we [[#Frontend Caddyfile|modified its Caddyfile previously]]:

```sh
systemctl --user restart caddy.service
```

### Set it up

You should _(hopefully)_ now be able to access your Vaultwarden instance.

![[Caddy#Debug|Caddy > Debug]]

You can now perform administrative tasks using the admin console, although you'd have to access it from the server directly, as per my [[#^c408d5|advanced Caddyfile]].

## Hardening

> [!warning]
> Always refer to up-to-date information and best practices and also consider reading up on the [official upstream Vaultwarden documentation](https://github.com/dani-garcia/vaultwarden/wiki/Hardening-Guide).
> The [[System Administration#Disclaimer|System Administration > Disclaimer]] applies here, too.

The file, we expand upon, is the [[#Frontend Caddyfile| > Frontend Caddyfile]], as the backend is simply the Vaultwarden container itself (served by Rocket internally).
The added/modified portions are highlighted, to enable quick expansion of an already existing (and hopefully working) `~/containers/caddy/config/Caddyfile` file:

```text title="~/containers/caddy/config/Caddyfile" {1-8,13-40}
# In combination with the `import admin_redir` statement, this only allows access to the admin interface from local networks
(admin_redir) {
	@admin {
		path /admin*
		not remote_ip private_ranges
	}
	redir @admin /
}

{$VAULTWARDEN_DOMAIN} {
	import subdomain-log {$VAULTWARDEN_DOMAIN}

	# This setting may have compatibility issues with some browsers
	# (e.g., attachment downloading on Firefox). Try disabling this
	# if you encounter issues.
	encode zstd gzip
	
	# Uncomment to improve security (WARNING: only use if you understand the implications!)
	# If you want to use FIDO2 WebAuthn, set X-Frame-Options to "SAMEORIGIN" or the Browser will block those requests
	header / {
		# Enable HTTP Strict Transport Security (HSTS)
		Strict-Transport-Security "max-age=31536000;"
		# Disable cross-site filter (XSS)
		X-XSS-Protection "0"
		# Disallow the site to be rendered within a frame (clickjacking protection)
		X-Frame-Options "SAMEORIGIN"
		# Prevent search engines from indexing (optional)
		X-Robots-Tag "noindex, nofollow"
		# Disallow sniffing of X-Content-Type-Options
		X-Content-Type-Options "nosniff"
		# Server name removing
		-Server
		# Remove X-Powered-By though this shouldn't be an issue, better opsec to remove
		-X-Powered-By
		# Remove Last-Modified because etag is the same and is as effective
		-Last-Modified
	}

	# Uncomment to allow access to the admin interface only from local networks
	import admin_redir

	# Proxy everything to Rocket
	reverse_proxy http://host.containers.internal:8000 {
		# Send the true remote IP to Rocket, so that Vaultwarden can put this in the
		# log, so that fail2ban can ban the correct IP.
		header_up X-Real-IP {remote_host}
	}
}
```
^c408d5

> [!todo] [[Caddy#Environment variables|Caddy > Environment variables]]
> `VAULTWARDEN_DOMAIN` : [[FQDN|FQDN]] of the Vaultwarden instance

You could in theory also [[Caddy#Don't terminate TLS|not terminate the TLS chain]].

### Disable registration

As you probably don't want _anyone_ to register an account uninvited, you should consider disabling registrations.
This preserves the _invite_ functionality.

You can either do that through the admin panel, or by setting `SIGNUPS_ALLOWED=false` in the [[Caddy#Environment variables|Caddy > Environment variables]].

### Disable password hints

To disable password hints, which can definitely compromise security, especially with non-random passwords (which you should of course **never** use), set `SHOW_PASSWORD_HINT=false` in the [[Caddy#Environment variables|Caddy > Environment variables]], or disable it using the admin panel.

### Redact token from logs

According to the [official hardening guide](https://github.com/dani-garcia/vaultwarden/wiki/Hardening-Guide#access-logs-contain-access_token-parameter), the `access_token` parameter should be redacted from logs.

You can do this within the `Caddyfile`.
Simply replace the `import subdomain-logs` line with the following snippet:

```text title="~/containers/caddy/config/Caddyfile
log {
	hostnames {$VAULTWARDEN_DOMAIN}
	output file /var/log/caddy/{$VAULTWARDEN_DOMAIN}.log
	
	format filter {
		wrap json
		fields {
			request>uri query {
				delete access_token
			}
			
			request>headers>Cookie cookie {
				replace session REDACTED
				delete secret
			}
		}
	}
}
```

### Rate limit login attempts

To prevent brute-force attacks, a rate limit at which login attempts can be made, should be employed.
After the limit is hit, you'd need to wait the specified time before being able to try again.

> [!info] MFA
> When using Multi-Factor-Authentication (or colloquially referred to as 2FA / Two-Factor-Authentication), the client _uses two requests_.
> 
> To match a value of _5 requests_ before the timeout is hit, I increased this value to 10.
> This should not be a problem, as the number of passwords an attacker would need to try to reliably brute-force my password (with an entropy of over 80 bits), is _much, much, much_ higher.

Add the parameters to the `~/containers/vaultwarden/vaultwarden.env` file.

```systemd title="~/containers/vaultwarden/vaultwarden.env"
LOGIN_RATELIMIT_MAX_BURST=10
LOGIN_RATELIMIT_SECONDS=60
ADMIN_RATELIMIT_MAX_BURST=10
ADMIN_RATELIMIT_SECONDS=60
```

### Fail2Ban

[[Fail2Ban (Fedora)|Install and set up Fail2Ban]].

First, try logging in with a random username and password and look for a line regarding the failed attempt within the log file `~/containers/vaultwarden/data/vaultwarden.log` (`$LOG_FILE`, specified in the [[#Environment file| > Environment file]]), akin to

```log title="~/containers/vaultwarden/data/vaultwarden.log"
[YYYY-MM-DD hh:mm:ss][vaultwarden::api::identity][ERROR] Username or password is incorrect. Try again. IP: XXX.XXX.XXX.XXX. Username: email@domain.com.
```

> [!NOTE] Log vs. Systemd
> As per the [official documentation on Fail2Ban](https://github.com/dani-garcia/vaultwarden/wiki/Fail2Ban-Setup), you could use `systemd-journal` instead of a log file.
> _However_, as Fail2Ban is installed at `root`-level, Fail2Ban would have a hard time quering the journal using `systemctl --user`.
> I found it more convenient to use a log file, as `root` can definitely read the user-owned file.

#### Filter

Create a new file `vaultwarden.local` under Fail2Ban's filter directory `/etc/fail2ban/filter.d`

```text title="/etc/fail2ban/filter.d/vaultwarden.local"
[INCLUDES]
before = common.conf

[Definition]
failregex = ^.*?Username or password is incorrect\. Try again\. IP: <ADDR>\. Username:.*$
ignoreregex =
```

#### Jail

Create a new file `vaultwarden.local` under Fail2Ban's jail directory `/etc/fail2ban/jail.d`

```text title="/etc/fail2ban/jail.d/vaultwarden.local"
[vaultwarden]
enabled = true
port = 80,443,1880,1443,8000,8080
filter = vaultwarden
logpath = /home/user/containers/vaultwarden/logs/vaultwarden.log
```

> [!todo] Replace
> `user` : username used for running the [[Podman (Fedora)#Rootless|rootless podman instance]].

#### SELinux

I ran into some problems with `fail2ban.service` not being able to read the log file, because of SELinux. 

This is a good thing. Normally.

To create policies for that, simply run the [[Fail2Ban (Fedora)#Restart|Fail2Ban (Fedora) > Restart]] command and immediately after check the output of `journalctl -xe`.

You should see a line containing the keywords `avc` and `denied`.
Simply copy this line and generate an SELinux policy:

```sh /PASTE YOUR LINE HERE/
audit2allow -M local << _EOF_
PASTE YOUR LINE HERE
_EOF_
```

Install it:

```sh
sudo semodule -i local.pp
```

I had to repeat this process once more, as another permission was missing, which means I ended up pasting _two audit lines_, before typing `_EOF_` to signify the end of input.

> [!question]- Use `systemd`?
> Fail2Ban is also capable of using `systemd` instead of log files.
> The only difficulty is that all my Systemd processes run in user mode.
> 
> I haven't yet found a solution of hooking into the user-specific Systemd journals.
> 
> It would probably be a cleaner way to do the tracking, though.

### Hiding under a subdirectory

> [!info]- I decided against it
> Although the [official hardening guide](https://github.com/dani-garcia/vaultwarden/wiki/Hardening-Guide#hiding-under-a-subdir) argues that this is not _security through obscurity_, but rather _defense in depth_, I would disagree.
> I don't find these two terms to be mutually exclusive.
> _Security through obscurity_ doesn't necessarily require it to be the _sole_ security measure, but rather describes a category of security measures.
> I, the same as many others, thoroughly disagree that _security through obscurity_ should be considered an effective security measure in our day and age.
> And although it might not hurt, I categorically try to avoid fooling myself on the effectiveness of employed measures, giving me a false sense of security.
> I'd rather focus on real and effective measures instead.
> 
> The way I see it, _defense in depth_ is already provided with my setup and adding another layer by using subdirectories.
> Using subdirectories, in my opinion, definitely qualifies as _security through obscurity_, adds no real security, is redundant, and confuses me more than a potential attacker.
> As it solely and specifically counts on an attacker not finding a _specific subdirectory_ the instance is hosted under, I could effectively use a randomly generated subdomain for nearly the same effect.
> I find the reliance on a non-diligent attacker in the day and age of completely automated attacks, port scans, botnets, etc., to be laughable, naive, and possibly ignorant.