---
{"publish":true,"created":"2025-05-02 10:47","tags":["OS/Fedora"],"cssclasses":""}
---


# Rootless Podman

## Data directories

First off, create directories in your home directory, where the Caddy configuration, data, and logs are supposed to be stored.

```sh
mkdir -p ~/containers/caddy/config
mkdir -p ~/containers/caddy/data
mkdir -p ~/containers/caddy/logs
```

We then also need to ensure the directory for storing the Podman Quadlet files is created

```sh
mkdir -p ~/.config/containers/systemd
```

### Static sites

If you want to serve a static side from within the Caddy container, also create a directory for each one, following the pattern

```sh /site-name(s)/
mkdir -p ~/containers/caddy/sites/{site-name(s)}
```

> [!todo] Replace
> - `site-name(s)` : a comma seperated list of directory names for the sites you intend to serve

If you don't, you could still consider simply creating the parent directory `~/containers/caddy/sites`, so you don't have to touch the generated [[Caddy#Quadlet file\|#Quadlet file]].
You would need to create corresponding entries in your [[Caddy#Caddyfile\|#Caddyfile]] anyway, for it to be active.

```sh
mkdir -p ~/containers/caddy/sites
```

## Ports

As we're running Podman rootless, the ports `80` (HTTP) and `443` (HTTPS) will certainly not be available.
There are numerous resolutions.
I intend to use this Caddy container to exclusively manage _every_ incoming (web) traffic, however.
That's why I simply decided to forward the ports `80` and `443` to a port my non-privileged user can get a hold of, namely `1880` and `1443`.

For that, first [[Firewalld\|install and enable firewalld]].

Subsequently, forward the corresponding ports

```sh
sudo firewall-cmd --permanent --add-forward-port=port=443:proto=tcp:toport=1443
sudo firewall-cmd --permanent --add-forward-port=port=80:proto=tcp:toport=1880
```

Reload firewalld

```sh
sudo firewall-cmd --reload
```

And check that everything went as planned

```sh
sudo firewall-cmd --list-all
```

## Caddyfile

The `Caddyfile` is used to describe the functionality of the Caddy instance. It is comparable to an `nginx` or `apache` config file.

The [official documentation](https://caddyserver.com/docs/) even provides a [list of common patterns](https://caddyserver.com/docs/caddyfile/patterns), like e.g., a static file server, or reverse proxying.

> [!example] Example: Reverse Proxy
> I mostly utilize this central Caddy instance for reverse proxying.
> 
> For example, I might have a second container running a web server on port `5000`.
> To serve it under the subdomain `service.dustvoice.de`, I would simply populate the Caddyfile under `~/containers/caddy/config/Caddyfile` with
> 
> ```text title="~/containers/caddy/config/Caddyfile" /service.dustvoice.de/ /5000/
> service.dustvoice.de {
> 	reverse_proxy localhost:5000
> }
> ```

### Environment variables

I often use environment variables to, for example, specify domains of my sub-sites (the sites this frontend Caddy instance proxies to), etc.
For this, I specify an `EnvironmentFile` in the [[Caddy#Quadlet file\|#Quadlet file]].

You can then specify environment variables within this file using the `NAME=val` pattern,
for example:

```sh title="~/containers/caddy/caddy.env"
APP_DOMAIN=app.domain.com
SECOND_APP_DOMAIN=second.domain.com
```

I would then use it in, e.g., my Caddyfile like so

```text title="~/containers/caddy/config/Caddyfile"
{$APP_DOMAIN} {
	reverse_proxy localhost:5000
}

{$SECOND_APP_DOMAIN} {
	reverse_proxy localhost:6000
}
```


> [!tip]
> If you don't want to use said environment file, you simply need to replace all occurences of a variable (`{$VAR_NAME}`) within the `Caddyfile` with the appropriate value.

### Logging

I usually insert a `subdomain-log` macro at the top of my `Caddyfile`s, to quickly enable logging within a subdomain section

```text title="~/containers/caddy/config/Caddyfile"
(subdomain-log) {
	log {
		hostnames {args[0]}
		output file /var/log/caddy/{args[0]}.log
	}
}
```

I can then simply do

```text title="~/containers/caddy/config/Caddyfile"
{$APP_DOMAIN} {
	import subdomain-log {$APP_DOMAIN}
}
```

## Quadlet file

We simply need to populate a `caddy.container` file within `~/.config/containers/systemd`:

```systemd title="~/.config/containers/systemd/caddy.container" /user/
[Unit]
Description=Caddy

[Container]
AutoUpdate=registry
ContainerName=caddy
Image=docker.io/library/caddy:latest
PublishPort=1880:80
PublishPort=1443:443
Volume=/home/user/containers/caddy/config:/etc/caddy:ro:Z
Volume=/home/user/containers/caddy/data:/data:Z
Volume=/home/user/containers/caddy/logs:/var/log/caddy:Z
Volume=/home/user/containers/caddy/sites:/srv:ro,z
EnvironmentFile=/home/user/containers/caddy/caddy.env

[Service]
Restart=always

[Install]
WantedBy=default.target
```

> [!info]- Some details
> - `PublishPort`
> 	- Map the outside ports `1880` and `1443`(see [[Caddy#Ports\|#Ports]]) to Caddy's internal `80` and `443` ports respectively
> 		- `-p 1880:80`
> 		- `-p 1443:443`
> - `Volume`
> 	- Map the outside [[Caddy#Data directories\|#Data directories]] to the correct internal paths
> 		- `-v ~/containers/caddy/config:/etc/caddy:ro,Z`
> 		- `-v ~/containers/caddy/data:/data:Z`
> 		- `-v ~/containers/caddy/logs:/var/log/caddy:Z`
> 		- `-v ~/containers/caddy/sites:/srv:ro,z` _(Remove this, if you would rather not serve static sites from this Caddy instance)_ ^677ece
> - `EnvironmentFile`
> 	- A file specifying environment variables available within the container following a `VAR_NAME=VALUE` pattern.
> 	- I often use this for specifying variables used in the [[Caddy#Caddyfile\|#Caddyfile]] using the `{$VAR_NAME}` syntax. Also see [[Caddy#Environment variables\|#Environment variables]]
> 
> > [!info] What are the `:z` and `:Z` labels?
> > These two labels are specific to [SELinux](https://www.redhat.com/en/topics/linux/what-is-selinux), which is enabled by default on Fedora.
> > Although some people might see it as an inconvenience, you shouldn't simply disable it, especially on a server, as it greatly hardens your system and increases security.
> > 
> > I found a good explanation elaborating these two options a bit in [this blog post](https://blog.ryanmartin.me/selinux-containers).

## Boot it up

### Reload

## Reload the daemon

As Quadlet files are `systemd` service files, you need to reload the daemon.

```sh
systemctl --user daemon-reload
```

This generates appropriate `.service` files.


### Start

## Start the service

```sh /name/
systemctl --user start name.service
```
 

> [!todo] Replace
> - `name` : `caddy`

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
> - `name` : `caddy`
### Linger

## Keep it running

As a rootless setup doesn't use a system-level service, all services would be stopped upon logout.

To prevent this, we must `enable-linger` (where `user` is your username, of course):

```sh /user/
loginctl enable-linger user
```


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


## Test it

Of course, you can proceed directly with, e.g., setting up a [[Nextcloud\|Nextcloud]] to test the Caddy setup.

I find it more convenient to perform a quick test, howevew, using a simple HTML file.

> [!NOTE]
> This assumes you [[Caddy#Static sites\|created the necessary directory]] and [[Caddy#^677ece\|configured Caddy to be able to serve static sites]], however.

You can easily create a crude `index.html` file in, for example, `~/containers/caddy/sites/test`

```html title="~/containers/caddy/sites/test/index.html"
<html>
	<head></head>
	<body>
		<p>Hello from Caddy!</p>
	</body>
</html>
```

and add a corresponding entry to your `Caddyfile`

```text title="~/containers/caddy/config/Caddyfile" /test.dustvoice.de/
test.dustvoice.de {
	root * /srv/test
	file_server
}
```

The site should now be accessible through the domain you specified and greet you with a `Hello from Caddy!`.

> [!warning] Consider removing it
> For security purposes, I would probably remove the `~/containers/caddy/sites/test` folder after testing.
> Removing the corresponding lines in your `Caddyfile` might be sufficient, but you most likely don't need to test it this way anytime soon again, so why clutter your system.

## Harden

### Don't terminate TLS

In the current scenario, the frontend caddy consumes TLS and proxies to the backend caddy or caddies.

This should be fine, if you trust the local network of your server, or at least your machine if you employed reasonable efforts of setting up, e.g., firewall rules, etc.

An alternative would be to also use TLS encryption between the frontend and backend instances.

The caddy community provides an [excellent post on how to do exactly that](https://caddy.community/t/use-caddy-for-local-https-tls-between-front-end-reverse-proxy-and-lan-hosts/11650).