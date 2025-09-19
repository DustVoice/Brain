---
{"publish":true,"aliases":"","created":"2025-05-30 14:57","modified":"2025-09-19T10:52:21.438+02:00","cssclasses":""}
---


There is no denying it, python is one of the most popular programming languages. And although I hate the syntax until the day I die (come at me, but indentation as syntax is a mistake), the shier number of packages, tools, libraries, APIs, yada, yada, makes it extremely flexible and versatile.

> [!quote]
> A jack of all trades is a master of none, but often times better than a master of one.

Or as I call it in German

> [!quote]
> Die Eierlegende Wollmilchsau

> [!info]
> For example for generating 3D models programmatically, [CadQuery](https://github.com/CadQuery/cadquery?tab=readme-ov-file) is the most viable contender for me, especially considering I don’t want to suffer from [OpenSCAD](https://openscad.org/) induced vomiting.
>
> Not giving up on my lispy dreams though, I’m trying out [Hy](https://hylang.org/) here and there, too.

## Install as a system package

### #OS/Fedora

> [!info]
> Should already be installed by default

```sh
sudo dnf install python
```

### System Packages vs. pip

Installing packages globally through `pip` isn’t the most optimal thing, especially when using it in a development environment.

It is [generally preferred](https://wiki.archlinux.org/title/Python#Arch_Repositories) to install the specific package through the official repositories or AUR, where the package name often follows the `python-<package_name>` scheme.

### Use a project manager

A project manager, like [[uv]] simplifies the development process immensely.

## Install in an environment

Another way is to utilize virtual environments.

I, personally, prefer using [miniforge](https://github.com/conda-forge/miniforge), which provides the `conda` and `mamba` commands, among others.

- `conda` is too bloated for me
- `anaconda` uses the _Anaconda Inc._ channel

I prefer the community supported `conda-forge` channel, though, which miniforge uses by default and exclusively. Furthermore, miniforge is installed in user space, so it marks a clear separation of a virtual environment to the system installation.

1. Download the latest installer script from the [download page](https://github.com/conda-forge/miniforge/releases/latest), or using `bash`

```sh
wget -O Miniforge3.sh "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-$(uname)-$(uname -m).sh"
```

2. Run the script through `bash`

```sh
bash Miniforge3.sh
```

3. Choose **no** when prompted whether you want to update your shell configuration or not (doesn’t work for all shells).
4. Add miniforge’s `bin` directory to your `$PATH`
5. Close and reopen your shell
6. `conda init`, if your shell supports it
7. `rm Miniforge3.sh` the installation script

> [!NOTE]
> If you're using my [[Dotfiles]] together with [[fish]], you don't have to let conda populate your init file, as the appropriate code is already included with them, upon detection of the `~/miniforge3`

Now when you wish to activate an environment, you can simply use

```sh /env_name/
conda activate env_name
```

## Additional Tools

You probably also want some additional tools to use with Python itself.

> [!todo] Install
> - [[ty]]
> - [[ruff]]
> - [[jedi]]
> - [[pylsp]]
