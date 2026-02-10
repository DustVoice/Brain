---
publish: true
aliases: ""
created: 2025-05-30 13:41
modified: 2026-02-11T00:31:32.579+01:00
cssclasses: ""
---


## Fedora

In this part, we're setting up _Fedora_ as our WSL distribution.

> [!info]
> Please also refer to [Fedora's official documentation](https://docs.fedoraproject.org/en-US/cloud/wsl/#_installing_fedora_in_wsl)

### Install

#### Microsoft Store

If you can use the _Microsoft Store_, installing the Fedora distribution is as easy as installing the _Fedora 42_ app from the store.

#### Command line

```ps
wsl.exe --install FedoraLinux-42
```

#### Tar-based

Download the tarball [from koji](https://koji.fedoraproject.org/koji/packageinfo?packageID=41688),
then check your WSL version

```ps
wsl.exe --version
```

> [!info]
> The following part is mostly copy/pasted from the [official documentation](https://docs.fedoraproject.org/en-US/cloud/wsl/#_installing_fedora_in_wsl).

##### Version >= 2.4.4

1. From the command line, install the tarball with `wsl --install --from-file .\path\to\Fedora.tar.xz`

```ps /VERSION/
wsl.exe --install --from-file .\Fedora-WSL-Base-VERSION.x86_64.tar.xz 
```

2. Enter the environment by running

```ps
wsl.exe -d Fedora
```

3. When prompted, provide a username. This will be the default user, and it will be added to the groups for `sudo` usage.

##### Version < 2.4.4

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

### Set a password

Make sure to set a password for the newly created user.
This should be self-explanatory from a security standpoint.

From within WSL, do

```sh /username/
sudo passwd username
```

### Set it as the default

If you don't have any other distro installed, you won't have to do anything.

If you do, however, and want to use the Fedora distro by default, just run the following

```ps
wsl.exe --set-default FedoraLinux-42
```

### Just Works™?

> [!note]
> This should only be neccessary if you’re behind a (corporate) proxy.

As I’m sitting behind a corporate `http` proxy, I initially had no access to the internet.

This can be confirmed by running

```sh
curl https://fedoraproject.org
```

To make WSL proxy-aware, I needed to

1. Add the exports to the `~/.bashrc`

```sh title="~/.bashrc" {1-11} /<hostname>/ /<port>/
PROXY_URL="http://<hostname>:<port>"
export all_proxy="$PROXY_URL"
export http_proxy="$PROXY_URL"
export https_proxy="$PROXY_URL"
export ftp_proxy="$PROXY_URL"
export no_proxy="127.0.0.1,0.0.0.0,localhost"
export ALL_PROXY="$PROXY_URL"
export HTTP_PROXY="$PROXY_URL"
export HTTPS_PROXY="$PROXY_URL"
export FTP_PROXY="$PROXY_URL"
export NO_PROXY="$no_proxy"
```

2. Source it

```sh
source ~/.bashrc
```

3. Allow `sudo` to pass these environment variables through by setting the `env_keep` property in `/etc/sudoers.d/proxy`

```text title="/etc/sudoers.d/proxy"
Defaults env_keep += "all_proxy http_proxy https_proxy ftp_proxy no_proxy ALL_PROXY HTTP_PROXY HTTPS_PROXY FTP_PROXY NO_PROXY"
```

Rerunning the `curl`-command should now produce a response, and your system should update just fine.

## NixOS

In this part, we're setting up _NixOS_ as our WSL distribution.

> [!info]
> Please also refer to the [Official NixOS-WSL documentation](https://github.com/nix-community/NixOS-WSL/releases/latest).

### Install

First download the latest `nixos.wsl` from the [NixOS-WSL GitHub release page](https://github.com/nix-community/NixOS-WSL/releases/latest).

Simply double click the file or run

```ps "nixos.wsl"
wsl.exe --install --from-file nixos.wsl
```

> [!todo] Replace
> - `nixos.wsl`: Path to your downloaded `nixos.wsl`

Afterwards you get dropped into NixOS.

> [!tip]
> When immediately bootstrapping a config like I do with my [[WSL 2 - Distro#Nix(OS) & Home-Manager config]], you don't need to update the system using channels (despite the welcome message telling you)!

### Set it as the default

If you don't have any other distro installed, you won't have to do anything.

If you do, however, and want to use the NixOS distro by default, just run the following

```ps
wsl.exe --set-default NixOS
```

## Nix(OS) & Home-Manager config

I have my own dendritic [nix-config](https://github.com/DustVoice/nix-config) that manages my complete system and home level configuration for different hosts, users, etc.

> [!tip]
> This completely replaces any manual configuration I previously did in my [[Dotfiles]] setup.

### Bootstrap

1. Rebuild the `boot` instead of directly `switch`ing to it

    ```shell
    sudo nixos-rebuild boot --flake github:DustVoice/nix-config#hostname
    ```

	^c10228

> [!todo] Replace
> - `hostname`: The machine's hostname, defined in [`modules/my/hosts`](https://github.com/DustVoice/nix-config/blob/main/modules/my/hosts.nix)

2. Exit the WSL shell and terminate the distro

    ```powershell
    wsl.exe -t NixOS
    ```

3. Start a shell as the `root` user and immediately exit, applying the new generation

    ```powershell
    wsl.exe -d NixOS --user root exit
    ```

4. Stop the distro again

    ```powershell
    wsl.exe -t NixOS
    ``` 

5. Finally open a WSL shell with (hopefully) everything applied.

> [!NOTE]
> This of course doesn't transfer any files from the `nixos` user's home directory to the newly created user!

### Just Works™?

> [!note]
> This should only be neccessary if you’re behind a (corporate) proxy.

As I’m sitting behind a corporate `http` proxy, I initially had no access to the internet.

This can be confirmed by running

```sh
curl https://nixos.org
```

Another symptom is that the [[WSL 2 - Distro#^c10228\|previous bootstrap command]] hangs indefinitely.

To circumvent this, we (redundantly) export local environment variables first:

```bash "PROXY-URL"
proxy_url="PROXY-URL"
export http_proxy="$proxy_url"
export https_proxy="$proxy_url"
export HTTP_PROXY="$proxy_url"
export HTTPS_PROXY="$proxy_url"
export CURL_NIX_FLAGS="-x $proxy_url"
```

Unfortunately there is only a limited set of environment variables which get copied over by `sudo`!
This might be wise from a security standpoint but is annoying in this case.
To circumvent this, add `--preserve-env=http_proxy,https_proxy,HTTP_PROXY,HTTPS_PROXY` as an argument to sudo:

> [!example] Modified bootstrap command
>
> ```sh
> sudo --preserve-env=http_proxy,https_proxy,HTTP_PROXY,HTTPS_PROXY,CURL_NIX_FLAGS nixos-rebuild switch --flake github:DustVoice/nix-config#hostname
> ```

> [!todo] Replace
> - `hostname`: The machine's hostname, defined in [`modules/my/hosts`](https://github.com/DustVoice/nix-config/blob/main/modules/my/hosts.nix)

This should run without problem and the proxy environment variables should be correctly set now and after reboots.
