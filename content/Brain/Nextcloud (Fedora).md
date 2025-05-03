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

> [!question]- Caddy within Caddy?
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

First, we tell the [[Nextcloud (Fedora)#^15ddf5|Caddy instance within the pod]] how to serve the Nextcloud by creating the necessary `Caddyfile` within `~/containers/nextcloud/caddy/config`:

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
                Strict-Transport-Security max-age=31536000;
        }
        
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

Therefore, we extend the appropriate [[./Caddy (Fedora)#Caddyfile|Caddy (Fedora) > Caddyfile]] under `~/containers/caddy/config/Caddyfile`

```sh title="~/containers/caddy/config/Caddyfile" /FQDN/
FQDN {
        redir /.well-known/carddav /remote.php/dav/ 301
        redir /.well-known/caldav /remote.php/dav/ 301

        header {
                Strict-Transport-Security max-age=31536000;
        }

        reverse_proxy http://host.containers.internal:8080
}
```

> [!todo] Replace
> `FQDN` (**F**ully **Q**ualified **D**omain **N**ame) : the (sub-)domain you intend to use for this instance (`subdomain.example.com`). 

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

As we configured a network for our [[Nextcloud (Fedora)#Pod|pod]], we will need to create the network, too.

```systemd title="~/.config/containers/systemd/nextcloud.network"
[Unit]
Description=Nextcloud Network

[Network]
Label=app=nextcloud
```

## Container
### Caddy

To set up the internal Caddy instance, which will use the [[Nextcloud (Fedora)#Caddyfile (internal)|previously created Caddyfile]], we simply create a new `~/.config/containers/systemd/nextcloud-caddy.container` file

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
> `user` : your username

This will also forward the _outside/system_ port `8080` to the _inside_ port `80`, specified in the [[Nextcloud (Fedora)#Caddyfile (internal)|aformentioned Caddyfile]].

### Database

Of course, Nextcloud requires a database. We'll use [MariaDB](https://mariadb.org), as it's one of the recommended choices for Nextcloud, [according to the official documentation](https://docs.nextcloud.com/server/latest/admin_manual/configuration_database/linux_database_configuration.html).
Apart from a performance standpoint, it doesn't really matter, since we won't _clutter_ our system by using a containerized approach.

#### Podman Secret

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
> `user` : your username
### Redis

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

### Nextcloud

Now we can finally create the _main Nextcloud container_.

Create the file under `~/.config/containers/systemd/nextcloud-app.container`

```systemd title="~/.config/containers/systemd/nextcloud-app.container" /user/ /FQDN/
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
AddHost=FQDN:127.0.0.1

[Install]
WantedBy=default.target
```

> [!todo] Replace
> - `user` : your username
> - `FQDN` : the fully qualified domain name (e.g., `subdomain.example.com`) you want to use for this instance
## Boot it up

![[./Podman (Fedora)#Reload the daemon|Podman (Fedora) > Reload the daemon]]

![[./Podman (Fedora)#Auto-Update|Podman (Fedora) > Auto-Update]]

![[./Podman (Fedora)#Keep it running|Podman (Fedora) > Keep it running]]

![[./Podman (Fedora)#Start the service|Podman (Fedora) > Start the service]]

> [!todo] Replace
> `name` : `nextcloud-pod`

![[./Podman (Fedora)#Check the status|Podman (Fedora) > Check the status]]

> [!todo] Replace
> `name` : `nextcloud-pod`
 
Look for _Started Nextcloud Pod_, to ensure Nextcloud has started successfully.
You can also check every other container's status by substituting `name` with the container's name.

Following that, you probably still need to restart the _external_ [[./Caddy (Fedora)|monolithic caddy instance]], as we [[Nextcloud (Fedora)#Caddyfile (internal)|modified its Caddyfile previously]]:

```sh
systemctl --user restart caddy.service
```

## Set it up

You should _(hopefully)_ now be able to access your Nextcloud installer under `https://FQDN.

> [!tip]
> If not, try checking the statuses of the `caddy`, `nextcloud-caddy` and `nextcloud-app` services.
> 
> You can also prepend an additional portion in front of all the content to the respective `Caddyfile`s, enabling more verbose error outputs.
> 
> ```text
> {
>         debug
> }
> ```

Choose a username for the admin account and generate a **(secure)** password, store it in your password manager and follow the installer.

## Remove the warnings

Most, if not all, of the warnings in your admin dashboard should go away after telling the Nextcloud what domains/proxies to trust.

First we enter the container

```sh
podman exec -it -u www-data nextcloud-app /bin/sh
```

Now we can use Nextcloud's `occ` tool

### Trust

> [!todo] Set environment variables
> - `$SERVER_IP` : your server's public IP
> - `$FQDN` : (sub-)domain you chose for this instance
> - `$REGION` : your region, for example, `DE`

```sh /SERVER_IP/ /FQDN/
php occ config:system:set trusted_domains 0 --value="nextcloud-caddy"
php occ config:system:set trusted_domains 1 --value="$SERVER_IP"
php occ	config:system:set trusted_domains 2 --value="$FQDN"
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

### Crontab

In order for the Nextcloud's crontab to be run regularly, we need to deploy a cronjob on the host side.

[[Cron (Fedora)#Install|Make sure you have crontab available]]

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

## Reboot

Finally, restart the Nextcloud, just for good measure. It should be lightning quick, too.

```sh
systemctl --user restart nextcloud-pod
```
