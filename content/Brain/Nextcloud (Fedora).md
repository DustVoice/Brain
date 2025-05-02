---
share: true
created: 2025-05-02 13:33
tags: 
---
# Using Podman (rootless)


> [!info] Source
> This section has been heavily inspired by [Micheal Jack](https://codeberg.org/mjack)'s [nextcloud-quadlet](https://codeberg.org/mjack/nextcloud-quadlets/src/branch/main) repository!
> I simply adapted his proceeding to my needs, by changing some paths, using my [[./Caddy (Fedora)|monolithic caddy instance]] for reverse proxying, etc.

^1bda1f


> [!NOTE] Caddy within Caddy?
> Note that, similar to the original author (see [[Nextcloud (Fedora)#^1bda1f|above]]), I use two Caddy instances.
> One to serve Nextcloud itself and another one for reverse proxying.
> 
> The only difference between my and the original author's approach is that my instance, responsible for reverse proxying, is not part of the Nextcloud pod.

^15ddf5

## Prerequisites

Make sure you have [[./Podman (Fedora)|podman]] installed and an outside [[./Caddy (Fedora)|monolithic caddy instance]] instance setup.

## Data directories

First off, create all the necessary directories:

```sh
mkdir -p ~/containers/nextcloud/{data,db,html,caddy/data,caddy/logs}
```

## Files

Next, we need to create the necessary files.

### Caddyfile (internal)

First, we tell the [[Nextcloud (Fedora)#^15ddf5|Caddy instance within the pod]] how to serve the Nextcloud by creating the necessary `Caddyfile` within `~/containers/nextcloud/caddy/`:

```text title="~/containers/nextcloud/web/Caddyfile"
:80 {
    root * /var/www/html
	file_server

	php_fastcgi nextcloud-app:9000

	redir /.well-known/carddav /remote.php/dav/ 301
	redir /.well-known/caldav /remote.php/dav/ 301

	# .htaccess / data / config / ... shouldn't be accessible from outside
	@forbidden {
			path    /.htaccess
			path    /data/*
			path    /config/*
			path    /db_structure
			path    /.xml
			path    /README
			path    /3rdparty/*
			path    /lib/*
			path    /templates/*
			path    /occ
			path    /console.php
	}

	respond @forbidden 404
}
```

This will serve the Nextcloud on the standard `80` HTTP port, **within the Nextcloud pod**.
The [[Nextcloud (Fedora)#Pod|pod]] configuration will take care of forwarding an actual _outside/system_ port to this _pod-internal_ one.

### Caddyfile (external)

As mentioned, the _external_ caddy instance is used for reverse proxying. Note that the _internal_ caddy instance expects incoming traffic on port `8080`, as specified in [[Nextcloud (Fedora)#Caddy|Caddy]].

Therefore, we extend the appropriate [[./Caddy (Fedora)#Caddyfile|Caddy (Fedora) > Caddyfile]] under `~/containers/caddy/config/Caddyfile`, of course, replacing `subdomain.domain` with the domain you intend to use for this instance.

```sh title="~/containers/caddy/config/Caddyfile" /subdomain.domain/
subdomain.domain {
    redir /.well-known/carddav /remote.php/dav/ 301
    redir /.well-known/caldav /remote.php/dav/ 301

    header {
        Strict-Transport-Security max-age=31536000;
    }

    reverse_proxy localhost:8080
}
```
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

As we configured a network for our [[Nextcloud (Fedora)#Pod|pod]], we will need to create the network, too.

```systemd title="~/.config/containers/systemd/nextcloud.network"
[Unit]
Description=Nextcloud Network

[Network]
Label=app=nextcloud
```


## Caddy

To set up the internal Caddy instance, which will use the [[Nextcloud (Fedora)#Caddyfile (internal)|previously created Caddyfile]], we simply create a new `~/.config/containers/systemd/nextcloud-caddy.container` file, replacing `user` with your username, of course

```systemd title="~/.config/containers/systemd/nextcloud-caddy.container" /user/
[Unit]
Description=Nextcloud Web
Wants=nextcloud-app.service
After=nextcloud-app.service

[Container]
Pod=nextcloud.pod
Label=app=nextcloud
AutoUpdate=registry
ContainerName=nextcloud-web
Image=docker.io/caddy:latest
Network=nextcloud.network
Volume=/home/user/containers/nextcloud/caddy/data:/data:Z
Volume=/home/user/containers/nextcloud/caddy/Caddyfile:/etc/caddy/Caddyfile:Z
Volume=/home/user/containers/nextcloud/caddy/logs:/var/log/caddy:Z
Volume=/home/user/containers/nextcloud/html:/var/www/html:ro,z
PublishPort=8080:80

[Install]
WantedBy=default.target
```

This will also forward the _outside/system_ port `8080` to the _inside_ port `80`, specified in the [[Nextcloud (Fedora)#Caddyfile (internal)|aformentioned Caddyfile]].

## Database

Of course, Nextcloud requires a database. We'll use [MariaDB](https://mariadb.org), as it's one of the recommended choices for Nextcloud, [according to the official documentation](https://docs.nextcloud.com/server/latest/admin_manual/configuration_database/linux_database_configuration.html).
Apart from a performance standpoint, it doesn't really matter, since we won't _clutter_ our system by using a containerized approach.

### Podman Secret

First, we generate a [Podman Secret]() to be used as the database password.

> [!warning]
> You should always **generate** this password!
> 
> Humans are not suitable password generators!

We use [[./pwgen (Fedora)|pwgen]] to generate a password and store it in a file, to **not** leak it to our shell history.

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

### Unit file

Create the `~/.config/containers/systemd/nextcloud-db.container` file, replacing `user` with your username, of course.

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

## Redis

For caching and other tasks, [redis](https://redis.io) is a pretty standard choice. I actually planned to use something different, but ended up using redis anyway, as I could simply copy [this container file](https://codeberg.org/mjack/nextcloud-quadlets/src/branch/main/quadlets/nextcloud-redis.container).

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

## App Container

Now we can finally create the _main Nextcloud container_.

Create the file under `~/.config/containers/systemd/nextcloud-app.container`, replacing `user` with your username, of course.

```systemd title="~/.config/containers/systemd/nextcloud-app.container" /user/
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

[Install]
WantedBy=default.target
```

## Boot it up

![[./Podman (Fedora)#Reload the daemon|Podman (Fedora) > Reload the daemon]]

![[./Podman (Fedora)#Auto-Update|Podman (Fedora) > Auto-Update]]

![[./Podman (Fedora)#Keep it running|Podman (Fedora) > Keep it running]]

![[./Podman (Fedora)#Start the service|Podman (Fedora) > Start the service]]
> 
> Replacing `name` with `nextcloud-pod`

![[./Podman (Fedora)#Check the status|Podman (Fedora) > Check the status]]
> 
> Again, replace `name` with `nextcloud-pod` and look for `Started Nextcloud Pod`
> You can also check every other container's status by substituting `name` with the container's name.

Following that, you probably still need to restart the _external_ [[./Caddy (Fedora)|monolithic caddy instance]], by using

```sh
systemctl --user restart caddy.service
```