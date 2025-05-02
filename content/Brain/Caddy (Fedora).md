---
share: true
created: 2025-05-02 10:47
tags: 
source: https://eshlox.net/run-the-caddy-server-on-fedora-using-podman
---

# Using Podman (rootless)

## Data directories

First off, create directories in your home directory, where the Caddy configuration, data, and logs are supposed to be stored.

```console
$ mkdir -p ~/containers/caddy/config
$ mkdir -p ~/containers/caddy/data
$ mkdir -p ~/containers/caddy/logs
```

We then also need to ensure the directory for storing the Podman Quadlet files is created

```console
$ mkdir -p .config/containers/systemd
```

If you want to serve a static side from within the Caddy container, also create a directory for each one, following the pattern

```console /<site-name>/
$ mkdir -p ~/containers/caddy/sites/<site-name>
```

> [!NOTE]
> If you would rather not touch the [[Caddy (Fedora)#Quadlet file|Quadlet file]] generated later, you should consider simply creating the parent directory `~/containers/caddy/sites`.
> If you don't populate the directory, and create corresponding entries in your [[Caddy (Fedora)#Caddyfile|Caddyfile]], it won't be active.

^cd4e46

## Ports

As we're running Podman rootless, the ports `80` (HTTP) and `443` (HTTPS) will certainly not be available.
There are numerous resolutions.
I intend to use this Caddy container to exclusively manage _every_ incoming (web) traffic, however.
That's why I simply decided to forward the ports `80` and `443` to a port my non-privileged user can get a hold of, namely `1880` and `1443`.

For that, first [[./Firewalld (Fedora)|install and enable firewalld]].

Subsequently, forward the corresponding ports

```console
$ sudo firewall-cmd --permanent --add-forward-port=port=443:proto=tcp:toport=1443
$ sudo firewall-cmd --permanent --add-forward-port=port=80:proto=tcp:toport=1880
```

Reload firewalld

```console
$ sudo firewall-cmd --reload
```

And check that everything went as planned

```console
$ sudo firewall-cmd --list-all
```

## Caddyfile

The `Caddyfile` is used to describe the functionality of the Caddy instance. It is comparable to an `nginx` or `apache` config file.

The [official documentation](https://caddyserver.com/docs/) even provides a [list of common patterns](https://caddyserver.com/docs/caddyfile/patterns), like e.g., a static file server, or reverse proxying.

> [!example] Reverse Proxy
> I mostly utilize this central Caddy instance for reverse proxying.
> 
> For example, I might have a second container running a web server on port `5000`.
> To serve it under the subdomain `service.dustvoice.de`, I would simply populate the Caddyfile with
> 
> ```text title="~/containers/config/Caddyfile /service.dustvoice.de/ /5000/
> service.dustvoice.de {
> 	reverse_proxy localhost:5000
> }
> ```

## Quadlet file

As mentioned earlier, `podlet` enables you to generate a Quadlet file for a specific `podman` command.

The `podman` command will
- Store the Quadlet file at the correct location
	- `--file ~/.config/containers/systemd/caddy.container`
- Run a new container with the name `caddy`
	- `podman run --name caddy`
- Try to restart the container, if it has stopped/crashed/…
	- `--restart always`
- Map the outside ports `1880` and `1443`(see [[Caddy (Fedora)#Ports|Ports]]) to Caddy's internal `80` and `443` ports respectively
	- `-p 1880:80`
	- `-p 1443:443`
- Map the outside [[Caddy (Fedora)#Data directories|Data directories]] to the correct internal paths
	- `-v ~/containers/caddy/config:/etc/caddy:ro,Z`
	- `-v ~/containers/caddy/data:/data:Z`
	- `-v ~/containers/caddy/logs:/var/log/caddy:Z`
	- `-v ~/containers/caddy/sites:/srv:ro,z` _(Remove this, if you would rather not serve static sites from this Caddy instance)_ ^677ece
- Use the latest caddy image available. **Don't use the `latest` tag!** Read up on [why you shouldn't use the latest tag](https://vsupalov.com/docker-latest-tag/)
	- `docker.io/library/caddy:2.10.0`

> [!info] What are the `:z` and `:Z` labels?
> These two labels are specific to [SELinux](https://www.redhat.com/en/topics/linux/what-is-selinux), which is enabled by default on Fedora.
> Although some people might see it as an inconvenience, you shouldn't simply disable it, especially on a server, as it greatly hardens your system and increases security.
> 
> I found a good explanation elaborating these two options a bit in [this blog post](https://blog.ryanmartin.me/selinux-containers).

This results in the following command:

```console
$ podlet --file ~/.config/containers/systemd/caddy.container --install --description Caddy podman run --name caddy --restart always -p 1880:80 -p 1443:443 -v ~/containers/caddy/config:/etc/caddy:ro,Z -v ~/containers/caddy/data:/data:Z -v ~/containers/caddy/logs:/var/log/caddy:Z -v ~/containers/caddy/sites:/srv:ro,z docker.io/library/caddy:2.10.0
```

It will produce something akin to (where `<user>` is your username, of course)

```systemd title="~/.config/containers/systemd/caddy.container" /<user>/
[Unit]
Description=Caddy

[Container]
ContainerName=caddy
Image=docker.io/library/caddy:2.10.0
PublishPort=1880:80
PublishPort=1443:443
Volume=/home/<user>/containers/caddy/config:/etc/caddy:ro:Z
Volume=/home/<user>/containers/caddy/data:/data:Z
Volume=/home/<user>/containers/caddy/logs:/var/log/caddy:Z
Volume=/home/<user>/containers/caddy/sites:/srv:ro,z

[Service]
Restart=always

[Install]
WantedBy=default.target
```

## Boot it up

First off, as Quadlet files are `systemd` service files, we need to reload the daemon.
As we're running it rootless, we need to specify the `--user` flag.

```console
$ systemctl --user daemon-reload
```

Now enable and start the service

```console
$ systemctl --user enable caddy
$ systemctl --user start caddy
```


> [!tip] Check the status
> You can check the status with either
> 
> ```console
> $ systemctl --user status caddy.service
> ```
> 
> or
> 
> ```console
> $ journalctl --user -xeu caddy.service
> ```

## Keep it running

As we didn't use a system-level service, but a rootless approach, all services would be stopped upon logout.

To prevent this, we must `enable-linger` (where `<user>` is your username, of course):

```console /<user>/
$ loginctl enable-linger <user>
```

## Test it

Of course, you can proceed with, e.g., setting up a [[./Nextcloud (Fedora)|Nextcloud (Fedora)]] to test the Caddy setup.

I find it more convenient to perform a quick test using a simple HTML file.

> [!NOTE]
> This assumes you [[Caddy (Fedora)#^cd4e46|created the necessary directory]] and [[Caddy (Fedora)#^677ece|configured Caddy to be able to serve static sites]], however.

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

```text title="~/containers/config/Caddyfile /dustvoice.de/
dustvoice.de {
	root * /srv/test
	file_server
}
```

The site should now be accessible through the domain you specified and greet you with a `Hello from Caddy!`.