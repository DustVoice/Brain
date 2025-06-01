---
{"publish":true,"created":"2025-05-02 13:33","cssclasses":""}
---




> [!danger]
> ## Proceed with caution, use at your own risk!
> 
> This is merely a documentation of my _specific setup_, i.e. what I found works _for me_.
> 
> You might have _entirely different_ requirements and expectations of _security_, etc.
> 
> ## Always use your **Brain™**
> 
> Always read up on _up-to-date_ documentation and _current_ best practices.
> Inform yourself, research, and treat my documentation as what it truly is: a mere info-dump.

# Rootless Podman

> [!info]- Source
> This section has been heavily inspired by [Micheal Jack](https://codeberg.org/mjack)'s [nextcloud-quadlet](https://codeberg.org/mjack/nextcloud-quadlets/src/branch/main) repository!
> I simply adapted his proceeding to my needs, by changing some paths, using my [[Caddy\|monolithic caddy instance]] for reverse proxying, etc.
^1bda1f

> [!question]- Caddy within Caddy?
> Note that, similar to the original author (see [[Nextcloud#^1bda1f\|above]]), I use two Caddy instances.
> One to serve Nextcloud itself and another one in front of it for reverse proxying.
> 
> The only difference between my and the original author's approach is that my instance, responsible for reverse proxying, is not part of the Nextcloud pod.
^15ddf5

## Prerequisites

Make sure you have [[Podman\|podman]] installed and a _frontend_ [[Caddy\|Caddy]] instance set up.

## Data directories

First off, create all the necessary directories:

```sh
mkdir -p ~/containers/nextcloud/{data,db,html,caddy/data,caddy/logs}
```

## Files

Next, we need to create the necessary files.

### Backend Caddyfile

First, we tell the [[Nextcloud#^15ddf5\|Caddy instance within the pod]] (which I'll refer to as the _backend_ instance), how to serve the Nextcloud by creating the necessary `Caddyfile` within `~/containers/nextcloud/caddy/config`:

```text title="~/containers/nextcloud/caddy/config/Caddyfile"
{
	servers {
		trusted_proxies static private_ranges
	}
}

:80 {
	root * /var/www/html
	file_server

	php_fastcgi nextcloud-app:9000

	redir /.well-known/carddav /remote.php/dav/ 301
	redir /.well-known/caldav /remote.php/dav/ 301

	header {
		Strict-Transport-Security "max-age=31536000;"
	}

	# .htaccess / data / config / ... shouldn't be accessible from outside
	@forbidden {
		path /.htaccess
		path /data/*
		path /config/*
		path /db_structure
		path /.xml
		path /README
		path /3rdparty/*
		path /lib/*
		path /templates/*
		path /occ
		path /console.php
	}

	respond @forbidden 404
}
```

This will
- serve the Nextcloud on the standard `80` HTTP port
- for the hostname equal to the name specified for its [[Nextcloud#Caddy\|container]]
- within the Nextcloud pod, or more specifically within it's specified [[Nextcloud#Network\|#Network]].

The [[Nextcloud#Pod\|pod configuration]] will take care of forwarding an actual _outside/system_ port to this _pod-internal_ one.

### Frontend Caddyfile

As mentioned, the _external_ caddy instance (which I will refer to as the _frontend_ instance) is used for reverse proxying. Note that the _backend_ caddy instance expects incoming traffic on port `8080`, as specified in [[Nextcloud#Caddy\|its container config]].

Therefore, we simply add a section to the (already present) [[Caddy#Caddyfile\|Caddy#Caddyfile]] under `~/containers/caddy/config/Caddyfile`

```sh title="~/containers/caddy/config/Caddyfile" /NEXTCLOUD_DOMAIN/
{$NEXTCLOUD_DOMAIN} {
	import subdomain-log {$NEXTCLOUD_DOMAIN}

	redir /.well-known/carddav /remote.php/dav/ 301
	redir /.well-known/caldav /remote.php/dav/ 301

	header {
		Strict-Transport-Security max-age=31536000;
	}

	reverse_proxy http://host.containers.internal:8080
}
```

> [!todo] [[Caddy#Environment variables\|Caddy#Environment variables]]
> `NEXTCLOUD_DOMAIN` : [[FQDN\|FQDN]] of the Nextcloud instance

> [!question]- How long did it take?
> Don't ask!
> 
> I won't admit how long it took me to realize, that my _Main Caddy_ container can't communicate with a _completely differnt caddy container, in a different pod, on a different network_ using `localhost:8080`.
> Thanks to [this excellent blog post](https://www.baeldung.com/linux/rootless-podman-communication-containers#host-port-publication), I was finally enlightened and released from my suffering.

## Pod

Create the `~/.config/containers/systemd/nextcloud.pod` file

```systemd title="~/.config/containers/systemd/nextcloud.pod"
[Unit]
Description=Nextcloud Pod

[Pod]
PodName=nextcloud
Network=nextcloud.network
```

## Network

As we configured a network for our [[Nextcloud#Pod\|#Pod]], we will need to create the network, too.

```systemd title="~/.config/containers/systemd/nextcloud.network"
[Unit]
Description=Nextcloud Network

[Network]
Label=app=nextcloud
```

## Containers
### Caddy

To set up the backend Caddy instance, which will use the [[Nextcloud#Backend Caddyfile\|previously created Caddyfile]], we simply create a new `~/.config/containers/systemd/nextcloud-caddy.container` file

```systemd title="~/.config/containers/systemd/nextcloud-caddy.container" /user/
[Unit]
Description=Nextcloud Web
Wants=nextcloud-app.service
After=nextcloud-app.service

[Container]
Pod=nextcloud.pod
Label=app=nextcloud
AutoUpdate=registry
ContainerName=nextcloud-caddy
Image=docker.io/caddy:latest
Network=nextcloud.network
Volume=/home/user/containers/nextcloud/caddy/data:/data:Z
Volume=/home/user/containers/nextcloud/caddy/config:/etc/caddy:Z
Volume=/home/user/containers/nextcloud/caddy/logs:/var/log/caddy:Z
Volume=/home/user/containers/nextcloud/html:/var/www/html:ro,z
PublishPort=8080:80

[Install]
WantedBy=default.target
```

> [!todo] Replace
> - `user` : username used for running the [[Podman#Rootless\|rootless podman instance]].

This will also forward the _outside/system_ port `8080` to the _inside_ port `80`, specified in the [[Nextcloud#Backend Caddyfile\|aformentioned Caddyfile]].

### Database

Of course, Nextcloud requires a database. We'll use [MariaDB](https://mariadb.org), as it's one of the recommended choices for Nextcloud, [according to the official documentation](https://docs.nextcloud.com/server/latest/admin_manual/configuration_database/linux_database_configuration.html).
Apart from a performance standpoint, it doesn't really matter, since we won't _clutter_ our system by using a containerized approach.

#### Podman Secret

First, we generate a [Podman Secret]() to be used as the database password.

> [!warning]
> You should always **generate** this password!
> 
> Humans are not suitable password generators!

We use [[pwgen\|pwgen]] to generate a password and store it in a file, to **not** leak it to our shell history.

```sh
pwgen -s 32 1 > pass.txt
```

This generates a single 32 character long password and stores it in `pass.txt`.

We can now generate the Podman secret with the name `nextcloud-mariadb-password`

```sh
podman secret create nextcloud-mariadb-password pass.txt
```

> [!warning] Delete the file
> Please remember to purge the password file afterwards!
> You can store the password securely in a password manager if you want, but you shouldn't have unencrypted plaintext passwords on your system.

#### Unit file

Create the `~/.config/containers/systemd/nextcloud-db.container` file

```systemd title="~/.config/containers/systemd/nextcloud-db.container" /user/
[Unit]
Description=Nextcloud Database

[Container]
Pod=nextcloud.pod
Label=app=nextcloud
AutoUpdate=registry
ContainerName=nextcloud-db
Image=docker.io/library/mariadb:10.11
Network=nextcloud.network
Volume=/home/user/containers/nextcloud/db:/var/lib/mysql:Z
Environment=MARIADB_RANDOM_ROOT_PASSWORD=1
Environment=MARIADB_AUTO_UPGRADE=1
Environment=MARIADB_DISABLE_UPGRADE_BACKUP=1
Environment=MYSQL_DATABASE=nextcloud
Environment=MYSQL_USER=nextcloud
Secret=nextcloud-mariadb-password,type=env,target=MYSQL_PASSWORD

[Install]
WantedBy=default.target
```

> [!todo] Replace
> - `user` : username used for running the [[Podman#Rootless\|rootless podman instance]].
### Redis

For caching and other tasks, [redis](https://redis.io) is a pretty standard choice. I actually planned to use [Valkey](https://valkey.io), but ended up using redis for now.
Simply enough, I simply copied [this container file](https://codeberg.org/mjack/nextcloud-quadlets/src/branch/main/quadlets/nextcloud-redis.container).

Create the file under `~/.config/containers/systemd/nextcloud-redis.container`:

```systemd title="~/.config/containers/systemd/nextcloud-db.container"
[Unit]
Description=Nextcloud Redis

[Container]
Pod=nextcloud.pod
Label=app=nextcloud
AutoUpdate=registry
ContainerName=nextcloud-redis
Image=docker.io/library/redis:alpine
Network=nextcloud.network

[Install]
WantedBy=default.target
```

### Nextcloud

Now we can finally create the _main Nextcloud container_.

Create the file under `~/.config/containers/systemd/nextcloud-app.container`

```systemd title="~/.config/containers/systemd/nextcloud-app.container" /user/ /NEXTCLOUD_DOMAIN/
[Unit]
Description=Nextcloud App
Wants=nextcloud-db.service nextcloud-redis.service
After=nextcloud-db.service nextcloud-redis.service

[Container]
Label=app=nextcloud
AutoUpdate=registry
Pod=nextcloud.pod
ContainerName=nextcloud-app
Image=docker.io/library/nextcloud:fpm-alpine
Network=nextcloud.network
Volume=/home/user/containers/nextcloud/data:/var/www/html/data:Z
Volume=/home/user/containers/nextcloud/html:/var/www/html/:Z
Environment=MYSQL_HOST=nextcloud-db
Environment=MYSQL_DATABASE=nextcloud
Environment=MYSQL_USER=nextcloud
Secret=nextcloud-mariadb-password,type=env,target=MYSQL_PASSWORD
Environment=REDIS_HOST=nextcloud-redis
AddHost=NEXTCLOUD_DOMAIN:127.0.0.1

[Install]
WantedBy=default.target
```

> [!todo] Replace
> - `user` : username used for running the [[Podman#Rootless\|rootless podman instance]].
> - `NEXTCLOUD_DOMAIN` : [[FQDN\|FQDN]] of the Nextcloud instance
## Boot it up

### Reload

## Reload the daemon

As Quadlet files are `systemd` service files, you need to reload the daemon.

```sh
systemctl --user daemon-reload
```

This generates appropriate `.service` files.


### Auto-Update

## Auto-Update

If you enabled the auto update feature using the `AutoUpdate` key in the `.container` file, you still need to enable the auto update timer

```sh
systemctl --user enable --now podman-auto-update.timer
```

### Linger

## Keep it running

As a rootless setup doesn't use a system-level service, all services would be stopped upon logout.

To prevent this, we must `enable-linger` (where `user` is your username, of course):

```sh /user/
loginctl enable-linger user
```


### Start

## Start the service

```sh /name/
systemctl --user start name.service
```


> [!todo] Replace
> - `name` : `nextcloud-pod`

### Status

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


> [!todo] Replace
> - `name` : `nextcloud-pod`
 
Look for _Started Nextcloud Pod_, to ensure Nextcloud has started successfully.
You can also check every other container's status by substituting `name` with the container's name.

### Restart

Following that, you probably still need to restart the _frontend_ [[Caddy\|Caddy]], as we [[Nextcloud#Frontend Caddyfile\|modified its Caddyfile previously]]:

```sh
systemctl --user restart caddy.service
```

## Set it up

You should _(hopefully)_ now be able to access your Nextcloud installer unde the domain you specified

### Debug

> [!tip]
> If something doesn't work right away, try checking the statuses of the `caddy`, and the (pod's)  container service(s).
> 
> You can also prepend an additional portion in front of all the content to the respective `Caddyfile`s, enabling more verbose error outputs.
> 
> ```text
> {
> 	debug
> }
> ```



Choose a username for the admin account and generate a **(secure)** password, store it in your password manager and follow the installer.

## Remove the warnings

Most, if not all, of the warnings in your admin dashboard should go away after telling the Nextcloud what domains/proxies to trust.

### Enter the container

First we enter the container

```sh
podman exec -it -u www-data nextcloud-app /bin/sh
```

Now we can use Nextcloud's `occ` tool

### Trust

> [!todo] Set environment variables
> - `$SERVER_IP` : your server's public IP
> - `$NEXTCLOUD_DOMAIN` : [[FQDN\|FQDN]] of this Nextcloud instance
> - `$REGION` : your region, for example, `DE`
> - `$CADDY` : hostname of your caddy container (in my guide it's `nextcloud-caddy`)

```sh /SERVER_IP/ /FQDN/
php occ config:system:set trusted_domains 0 --value="$CADDY"
php occ config:system:set trusted_domains 1 --value="$SERVER_IP"
php occ	config:system:set trusted_domains 2 --value="$NEXTCLOUD_DOMAIN"
php occ	config:system:set trusted_proxies 0 --value="$SERVER_IP"
php occ config:system:set overwrite.cli.url --value "https://$FQDN"
php occ config:system:set overwriteprotocol --value "https"
php occ config:system:set default_phone_region --value "$REGION"
php occ config:system:set proxyexclude 1 --value="localhost"
php occ config:system:set proxyexclude 2 --value="127.0.0.1
```

### Mime type migrations

```sh
php occ maintenance:repair --include-expensive
```

### Add missing indices

```sh
php occ db:add-missing-indices
```

### Set maintenance window

Some maintenance tasks only run once a day.
To prevent them from being run during the main usage time, we can set the start of the maintenance window, as per the [official documentation](https://docs.nextcloud.com/server/31/admin_manual/configuration_server/background_jobs_configuration.html#parameters):

```sh /value/
php occ config:system:set maintenance_window_start --type=integer --value=1
```


> [!info] `value`
> The above value for `value` of `1`, means that the aforementioned background job will only be run between _01:00am UTC_ and _05:00am UTC_.


## Crontab

In order for the Nextcloud's crontab to be run regularly, we need to deploy a cronjob on the host side.

Make sure, you have the `crontab` command available, by installing [[Cronie#OS/Fedora\|Cronie#OS/Fedora]].

```sh
crontab -e
```

then paste in the cronjob:

```text
*/5 * * * * podman  exec -t -u www-data nextcloud-app php -f /var/www/html/cron.php
```

Save it and check if everything went smoothly

```sh
crontab -l
```

## Hardening

Security should be more than fine, by using rootless containers (even for the reverse proxy caddy), isolating the network, etc.
Still, security is always a concern and should be one of the top priorities.

As always, though, always refer to up-to-date information and best practices and also consider reading up on the [official upstream Nextcloud documentation](https://docs.nextcloud.com/server/31/admin_manual/installation/harden_server.html).
The [[System Administration#Disclaimer\|System Administration#Disclaimer]] applies here, too.

I have collected a couple of additional options for the [[Nextcloud#Backend Caddyfile\|#Backend Caddyfile]] that should harden the instance even more.
Most of these options aim at _future-proofing_ the installation and, for example, prevent access to files which _should_ be unproblematic, but _might_ not be (in the future).
If you encounter weird problems or issues, it might be related to too restrictive of a config, so you might need to experiment with the introduced options, to determine which caused the error.

The file, we expand upon, is the [[Nextcloud#Backend Caddyfile\|#Backend Caddyfile]], as the [[Nextcloud#Frontend Caddyfile\|frontend one]] solely describes the reverse proxy behavior.
The added/modified portions are highlighted, to enable quick expansion of an already existing (and hopefully working) `~/containers/nextcloud/caddy/config/Caddyfile` file:

```text title="~/containers/nextcloud/caddy/config/Caddyfile" /includeSubDomains;/ /preload/ {19-32,35-87,92,95-96,98,101,103-104,109} 
{
	servers {
		trusted_proxies static private_ranges
	}
}

:80 {
	root * /var/www/html
	file_server

	php_fastcgi nextcloud-app:9000

	redir /.well-known/carddav /remote.php/dav/ 301
	redir /.well-known/caldav /remote.php/dav/ 301

	header {
		Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"

		# More security hardening headers
		Referrer-Policy "no-referrer"
		X-Content-Type-Options "nosniff"
		X-Download-Options "noopen"
		X-Frame-Options "SAMEORIGIN"
		X-Permitted-Cross-Domain-Policies "none"
		X-Robots-Tag "noindex, nofollow"
		X-XSS-Protection "1; mode=block"
		# Permissions-Policy "interest-cohort=()"

		# Remove X-Powered-By header, which is an information leak
		-X-Powered-By
		# Replace http with https in any Location header
		Location http:// https://
	}

	# Cache control
	@static {
		file
		path *.css *.js *.svg *.gif
	}

	header @static {
		Cache-Control "max-age=360"
	}

	@fonts {
		path /core/fonts
	}

	header @fonts {
		Cache-Control "max-age=604800"
	}

	# gzip encoding
	encode {
		gzip 4
		minimum_length 256

		match {
			header Content-Type application/atom+xml*
			header Content-Type application/javascript*
			header Content-Type application/json*
			header Content-Type application/ld+json*
			header Content-Type application/manifest+json*
			header Content-Type application/rss+xml*
			header Content-Type application/vnd.geo+json*
			header Content-Type application/vnd.ms-fontobject*
			header Content-Type application/x-font-ttf*
			header Content-Type application/x-web-app-manifest+json*
			header Content-Type application/xhtml+xml*
			header Content-Type application/xml*
			header Content-Type font/opentype*
			header Content-Type image/bmp*
			header Content-Type image/svg+xml*
			header Content-Type image/x-icon*
			header Content-Type application/atom+xmlapplication/javascript*
			# Would this be a good idea?
			header Content-Type text/*
			# header Content-Type text/cache-manifest*
			# header Content-Type text/css*
			# header Content-Type text/plain*
			# header Content-Type text/vcard*
			# header Content-Type text/vnd.rim.location.xloc*
			# header Content-Type text/vtt*
			# header Content-Type text/x-component*
			# header Content-Type text/x-cross-domain-policy*
		}
	}

	# .htaccess / data / config / ... shouldn't be accessible from outside
	@forbidden {
		path /.htaccess
		path /.user.ini
		path /.xml
		path /3rdparty/*
		path /autotest
		path /build/*
		path /config/*
		path /console
		path /console.php
		path /data/*
		path /db_
		path /db_structure
		path /indie
		path /issue
		path /lib/*
		path /occ
		path /README
		path /templates/*
		path /tests/*
	}

	respond @forbidden 404
}
```

Of course, you need to at least restart the `nextcloud-caddy.service` if you changed this file after the [[Nextcloud#Reboot\|#Reboot]] step.

You could in theory also [[Caddy#Don't terminate TLS\|not terminate the TLS chain]].
## Reboot

Finally, restart the Nextcloud, just for good measure. It should be lightning quick, too.

```sh
systemctl --user restart nextcloud-pod
```
