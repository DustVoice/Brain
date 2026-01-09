---
{"publish":true,"created":"2025-09-19","modified":"2025-12-12T18:37:36.762+01:00","cssclasses":""}
---


> [!cite]
> Your new coding bestie, now available in your favourite terminal.
> Your tools, your code, and your workflows, wired into your LLM of choice.
>
> -- [GitHub Repo](https://github.com/charmbracelet/crush)

## Install

### [[Fedora]]

```sh
echo '[charm]
name=Charm
baseurl=https://repo.charm.sh/yum/
enabled=1
gpgcheck=1
gpgkey=https://repo.charm.sh/yum/gpg.key' | sudo tee /etc/yum.repos.d/charm.repo
sudo dnf install crush
```

### [[Homebrew]]

```sh
brew install charmbracelet/tap/crush
```

## Configure

Follow the [Getting Started](https://github.com/charmbracelet/crush#getting-started) section of the project's README.
Essentially create a config, or set an API key environment variable.
