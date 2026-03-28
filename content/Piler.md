---
publish: true
created: 2025-05-02 13:33
---


![[Disclaimer (Tech)]]

## Rootless Podman

### Prerequisites

Make sure you have [[Podman\|podman]] installed and a _frontend_ [[Caddy]] instance set up.

### Data directories

First off, create all the necessary directories:

```sh
mkdir -p ~/containers/piler/{app/etc,app/store}
mkdir -p ~/containers/piler/{db/conf,db/data}
mkdir -p ~/containers/piler/{manticore/conf,manticore/data}
```

### Frontend Caddyfile

As mentioned, the _external_ caddy instance (which I will refer to as the _frontend_ instance) is used for reverse proxying. Note that Piler expects incoming (`HTTP`) traffic on port `8180`, as specified in [[Piler#Pod\|its pod file]].

Therefore, we simply add a section to the (already present) [[Caddy#Caddyfile]] under `~/containers/caddy/config/Caddyfile`

```sh title="~/containers/caddy/config/Caddyfile" {1-9} /PILER_DOMAIN/
{$PILER_DOMAIN} {
	import subdomain-log {$PILER_DOMAIN}
	
	reverse_proxy http://host.containers.internal:8180 {
			# Send the true remote IP to Piler.
			# This enables fail2ban to ban the correct IP.
			header_up X-Real-IP {remote_host}
	}
}
```

> [!todo] [[Caddy#Environment variables]]
> `PILER_DOMAIN` : [[FQDN]] of the Piler instance

### Pod

Create the `~/.config/containers/systemd/piler.pod` file

```systemd title="~/.config/containers/systemd/piler.pod"
[Unit]
Description=Piler Pod

[Pod]
PodName=piler
Network=piler.network
PublishPort=8125:25
PublishPort=8180:80
```

### Ports

As you can see, we just published the container ports `80` and `25` to the system ports `8180` and `8125` respectively.

The [[Piler#Frontend Caddyfile\|frontend Caddy config]] proxies HTML traffic to the `8180` port, but we still need to open `8125`, more precisely, _forward_ the system level privileged port `25` to the unprivileged port used by the rootless podman `8125`.

```sh
sudo firewall-cmd --permanent --add-forward-port=port=25:proto=tcp:toport=8125
```

Reload the Firewall

```sh
sudo firewall-cmd --reload
```

And check that everything went as planned

```sh
sudo firewall-cmd --list-all
```

### Network

As we configured a network for our [[Piler#Pod]], we will need to create the network, too.

```systemd title="~/.config/containers/systemd/piler.network"
[Unit]
Description=Piler Network

[Network]
Label=app=piler
DisableDNS=false
Internal=false
```

### Containers

#### Database

Of course, Piler requires a database. We'll use [MariaDB](https://mariadb.org), as it's the default, per the [official `docker-compose.yaml`](https://github.com/jsuto/piler/blob/master/docker/docker-compose.yaml)

##### Podman Secret

First, we generate a [Podman Secret]() to be used as the database password.

> [!warning]
> You should always **generate** this password!
>
> Humans are not suitable password generators!

We use [[pwgen\|pwgen]] to generate a password and store it in a file, to **not** leak it to our shell history.

We can now generate the Podman secret with the name `piler-mariadb-password`

```sh
echo -n $(pwgen -s 32 1) | podman secret create piler-mariadb-password -
```

This generates a single 32 character long password.

> [!note]
> The `-n` part is needed, to en sure, that there isn't a newline present in the secret.

> [!tip]
> If you want to store the password securely in a password manager, you can simply retrieve the secret using
>
> ```sh
> podman secret inspect piler-mariadb-password --showsecret
> ```

##### Config File

Populate MariaDB's default config file under `~/containers/piler/db/conf/piler.cnf`, according to the [upstream file](https://github.com/jsuto/piler/blob/master/docker/piler.cnf):

```txt title="~/containers/piler/db/conf/piler.cnf"
[mariadb]

innodb_buffer_pool_size = 256M
innodb_flush_log_at_trx_commit=1
innodb_log_buffer_size=64M
innodb_log_file_size=64M
innodb_read_io_threads=4
innodb_write_io_threads=4

innodb_file_per_table
```

##### Environment File

This [specifies environment variables available to the container](https://github.com/dani-garcia/vaultwarden/wiki/Enabling-admin-page#secure-the-admin_token).

Create and initially populate the `db.env` file under the [[Piler#Data directories\|previously created directory]] `~/containers/piler/db`

```systemd title="~/containers/piler/db/db.env"
MYSQL_DATABASE=piler
MYSQL_USER=piler
MYSQL_RANDOM_ROOT_PASSWORD=yes
```

##### Container File

Create the `~/.config/containers/systemd/piler-db.container` file

```systemd title="~/.config/containers/systemd/piler-db.container" /user/
[Unit]
Description=Piler Database

[Container]
Pod=piler.pod
Label=app=piler
ContainerName=piler-db
AutoUpdate=registry
Image=docker.io/library/mariadb:11.6.2
AddCapability=chown dac_override setuid setgid
DropCapability=ALL
EnvironmentFile=/home/user/containers/piler/db/db.env
Secret=piler-mariadb-password,type=env,target=MYSQL_PASSWORD
Exec='--character-set-server=utf8mb4' '--collation-server=utf8mb4_unicode_ci'
Volume=/home/user/containers/piler/db/data:/var/lib/mysql:Z
Volume=/home/user/containers/piler/db/conf:/etc/mysql/conf.d:ro,Z

[Install]
WantedBy=default.target
```

> [!todo] Replace
> - `user` : username used for running the [[Podman#Rootless\|rootless podman instance]].

#### Memcached

For caching, we use `memcached`.

Create the file under `~/.config/containers/systemd/piler-memcached.container`:

```systemd title="~/.config/containers/systemd/piler-memcached.container"
[Unit]
Description=Piler Memcached

[Container]
Pod=piler.pod
Label=app=piler
AutoUpdate=registry
ContainerName=piler-memcached
Image=docker.io/library/memcached:latest
Exec=-m 64

[Install]
WantedBy=default.target
```

#### Manticore

To provide a good search experience, Piler uses Manticore.

##### Config File

> [!caution] Config file name
> Normally, the filename for the config should be `manticore.conf`.
> I, however, was unable to get it up and running.
>
> Upon further investigation, it seems as if the [docker image](https://hub.docker.com/layers/manticoresearch/manticore/latest/images/sha256-638de83aa45fde32b0af7e359cce6620edb8effd95eb266e347c1fbb95e7da23) is faulty, in that the `searchd` parameter defining the config file is hard-coded to `manticore.conf.sh`.
>
> So for now, I chose to simply rename the file accordingly.

Populate Manticore's config file under `~/containers/piler/manticore/conf/manticore.conf.sh`, according to the [upstream file](https://github.com/jsuto/piler/blob/master/docker/manticore.conf):

```text title="~/containers/piler/manticore/conf/manticore.conf.sh"
## Feel free to customize the config, eg. set the charset_table to your choice, etc.

index piler1
{
    type = rt
    path = /var/lib/manticore/piler1
    rt_mem_limit = 512M
    stored_fields =
    min_word_len = 1
    min_prefix_len = 5
    #charset_table  = 0..9, english, _,
    # See https://manual.manticoresearch.com/Creating_an_index/Data_types#Row-wise-and-columnar-attribute-storages
    # if you want to enable columnar storage
    # columnar_attrs = *
    rt_field = sender
    rt_field = rcpt
    rt_field = senderdomain
    rt_field = rcptdomain
    rt_field = subject
    rt_field = body
    rt_field = attachment_types
    rt_attr_bigint = arrived
    rt_attr_bigint = sent
    rt_attr_uint = size
    rt_attr_uint = direction
    rt_attr_uint = folder
    rt_attr_uint = attachments
}

index tag1
{
    type = rt
    path = /var/lib/manticore/tag1
    rt_mem_limit = 16M
    stored_fields = tag
    min_word_len = 2
    min_prefix_len = 5
    #charset_table  = 0..9, english, _,
    rt_field = tag
    rt_attr_bigint = mid
    rt_attr_uint = uid
}

index note1
{
    type = rt
    path = /var/lib/manticore/note1
    rt_mem_limit = 16M
    stored_fields = note
    min_word_len = 2
    min_prefix_len = 5
    #charset_table  = 0..9, english, _,
    rt_field = note
    rt_attr_bigint = mid
    rt_attr_uint = uid
}

index audit1
{
    type = rt
    path = /var/lib/manticore/audit1
    rt_mem_limit = 16M
    stored_fields = *
    min_word_len = 2
    min_prefix_len = 5
    #charset_table  = 0..9, english, _,
    rt_field = email
    rt_field = ipaddr
    rt_field = description
    rt_attr_bigint = ts
    rt_attr_bigint = meta_id
    rt_attr_uint = action
}

searchd
{
    listen                  = piler-manticore:9312
    listen                  = piler-manticore:9306:mysql
    listen                  = piler-manticore:9307:mysql_readonly
    log                     = /var/lib/manticore/manticore.log
    binlog_max_log_size     = 256M
    binlog_path             = /var/lib/manticore
    binlog_flush            = 2
    query_log               = /var/lib/manticore/query.log
    network_timeout         = 5
    pid_file                = /var/lib/manticore/manticore.pid
    seamless_rotate         = 1
    preopen_tables          = 1
    unlink_old              = 1
    thread_stack            = 512k
    # https://manticoresearch.com/blog/manticoresearch-buddy-intro/
    # Give a value to the buddy_path variable to enable manticore-buddy
    buddy_path              =
    rt_flush_period         = 300
}
```

##### Container File

Create the file under `~/.config/containers/systemd/piler-manticore.container`:

```systemd title="~/.config/containers/systemd/piler-manticore.container" /user/
[Unit]
Description=Piler Manticore

[Container]
Pod=piler.pod
Label=app=piler
AutoUpdate=registry
ContainerName=piler-manticore
Image=docker.io/manticoresearch/manticore:9.3.2
User=manticore
Volume=/home/user/containers/piler/manticore/conf:/etc/manticoresearch:Z
Volume=/home/user/containers/piler/manticore/data:/var/lib/manticore:Z

[Install]
WantedBy=default.target
```

> [!todo] Replace
> - `user` : username used for running the [[Podman#Rootless\|rootless podman instance]].

#### Piler

Now we can finally create the _main Piler container_.

##### Environment File

This [specifies environment variables available to the container](https://github.com/dani-garcia/vaultwarden/wiki/Enabling-admin-page#secure-the-admin_token).

Create and initially populate the `app.env` file under the [[Piler#Data directories\|previously created directory]] `~/containers/piler/app`

```systemd title="~/containers/piler/app/app.env"
MANTICORE_HOSTNAME=piler-manticore
MEMCACHED_HOSTNAME=piler-memcached
MYSQL_HOSTNAME=piler-db
MYSQL_DATABASE=piler
MYSQL_USER=piler
MYSQL_USERNAME=piler
PILER_HOSTNAME=PILER_DOMAIN
RT=1
```

> [!todo] Replace
> - `PILER_DOMAIN` : [[FQDN]] of the Piler instance

##### Container File

Create the file under `~/.config/containers/systemd/piler-app.container`

```systemd title="~/.config/containers/systemd/piler-app.container" /user/ /PILER_DOMAIN/
[Unit]
Description=Piler App
Wants=piler-db.service piler-memcached.service piler-manticore.service
After=piler-db.service piler-memcached.service piler-manticore.service

[Container]
Label=app=piler
AutoUpdate=registry
Pod=piler.pod
ContainerName=piler-app
Image=docker.io/sutoj/piler:1.4.7b
EnvironmentFile=/home/user/containers/piler/app/app.env
Secret=piler-mariadb-password,type=env,target=MYSQL_PASSWORD
HealthCmd="curl -s smtp://localhost/"
HealthInterval=20s
HealthRetries=3
HealthStartPeriod=15s
HealthTimeout=3s
RunInit=true
Volume=/home/user/containers/piler/app/etc:/etc/piler:Z
Volume=/home/user/containers/piler/app/store:/var/piler/store:Z

[Service]
MemoryMax=512M

[Install]
WantedBy=default.target
```

> [!todo] Replace
> - `user` : username used for running the [[Podman#Rootless\|rootless podman instance]].

### Boot it up

#### Reload

![[Podman#Reload the daemon]]

#### Auto-Update

![[Podman#Auto-Update]]

#### Linger

![[Podman#Keep it running]]

#### Start

![[Podman#Start the service]]

> [!todo] Replace
> - `name` : `piler-pod`

#### Status

![[Podman#Check the status]]

> [!todo] Replace
> - `name` : `piler-pod`

#### Restart

Following that, you probably still need to restart the _frontend_ [[Caddy]], as we [[Piler#Frontend Caddyfile\|modified its Caddyfile previously]]:

```sh
systemctl --user restart caddy.service
```

### Set it up

You should _(hopefully)_ now be able to access your Piler instance under the [[FQDN]] you specified.

The default login data is:

| Username    | Password   |
| ----------- | ---------- |
| admin@local | pilerrocks |
