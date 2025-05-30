---
share: true
created: 2025-05-30 13:41
tags: 
---
# Fedora

In this part, we're setting up _Fedora_ as our WSL distribution.

> [!info]
> Please also refer to [Fedora's official documentation](https://docs.fedoraproject.org/en-US/cloud/wsl/#_installing_fedora_in_wsl)

## Install
### Microsoft Store

If you can use the _Microsoft Store_, installing the Fedora distribution is as easy as installing the _Fedora 42_ app from the store.

### Command line

```ps
wsl.exe --install FedoraLinux-42
```

### Tar-based

Download the tarball [from koji](https://koji.fedoraproject.org/koji/packageinfo?packageID=41688,
then check your WSL version

```ps
wsl.exe --version
```


> [!info]
> The following part is mostly copy/pasted from the [official documentation](https://docs.fedoraproject.org/en-US/cloud/wsl/#_installing_fedora_in_wsl).

#### Version >= 2.4.4

1. From the command line, install the tarball with `wsl --install --from-file .\path\to\Fedora.tar.xz`
```ps /VERSION/
wsl.exe --install --from-file .\Fedora-WSL-Base-VERSION.x86_64.tar.xz 
```
2. Enter the environment by running
```ps
wsl.exe -d Fedora
```
3. When prompted, provide a username. This will be the default user, and it will be added to the groups for `sudo` usage.

#### Version < 2.4.4

(These steps assume you are using PowerShell)

1. Make a directory for the Fedora distribution with
```ps
mkdir $ENV:LOCALAPPDATA\WSL\Fedora
```
2. Import the WSL tarball with
```ps /VERSION/
wsl.exe --import Fedora $ENV:LOCALAPPDATA\WSL\Fedora .\Fedora-WSL-Base-VERSION.x86_64.tar.xz 
```
3. Enter the environment with
```ps
wsl.exe -d Fedora -u root
```
4. Manually run `/usr/libexec/wsl/oobe.sh` to create the default user
5. `exit` the environment logged in as root
6. Enter the environment as the newly created user with 
```ps /username/
wsl.exe -d Fedora -u username
```

## Set a password

Make sure to set a password for the newly created user.
This should be self-explanatory from a security standpoint.

From within WSL, do

```sh /username/
sudo passwd username
```

# Just Works™?

> [!note]
> This should only be neccessary if you’re behind a (corporate) proxy.

As I’m sitting behind a corporate `http` proxy, I initially had no access to the internet.

This can be confirmed by running

```sh
curl https://fedoraproject.org
```

To make WSL proxy-aware, I needed to

1. Add the exports to the `~/.bashrc`
```sh title="~/.bashrc" {1-3}
export http_proxy=http://<hostname>:<port>
export https_proxy=$http_proxy
export ftp_proxy=$http_proxy
```
2. Source it
```sh
source ~/.bashrc
```
3. Allow `sudo` to pass these environment variables through by setting the `env_keep` property in `/etc/sudoers.d/proxy`
```text title="/etc/sudoers.d/proxy"
Defaults env_keep += "http_proxy https_proxy ftp_proxy"
```

Rerunning the `curl`-command should now produce a response, and your system should update just fine.