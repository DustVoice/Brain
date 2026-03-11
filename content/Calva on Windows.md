---
publish: true
created: 2025-09-19
modified: 2026-03-11T15:57:44.818+01:00
tags:
  - OS/Windows
cssclasses: ""
---

> [!note]
> This loose guide is aimed to assist my coworkers in setting up Calva (and the clojure development environment) on Windows!

## Adoptium

To install a JDK that doesn't suck (if you don't have a JDK already installed), go to https://adoptium.net/download to download & install Adoptium.

> [!important]
> Make sure you **enable** the option to set the `JAVA_HOME` variable in the installer!

## Clojure

To install clojure and accompanying tools, we'll use [clj-msi](https://github.com/casselc/clj-msi), which itself uses [deps.clj](https://github.com/borkdude/deps.clj), which is a great rewrite of the _official_ Clojure CLI Tools written in Bash (urgh!).

Simply download the latest `.msi` installer from the [Release page](https://github.com/casselc/clj-msi/releases) and install the software.

## Git

Both [[Calva on Windows#Calva]], as well as [[Calva on Windows#Clojure]] expect Git for several operations.

The simplest way to install is by using

```powershell
winget.exe install Git.Git
```

from a PowerShell prompt.

Go through the installation and restart any applications, as `git` needs to be in the `$PATH` and most programs only inherit the `PATH` environment variable at launch.

## Proxy

If you are sitting behind a corporate proxy, you need to configure the proxy for Clojure (Maven to be exact) first.
They both use the same centralized `.\m2\settings.xml` config file for that.
Under Windows the config file should be (created) under
- For `cmd.exe`
	- `%USERPROFILE%\.m2\settings.xml`
- For PowerShell
	- `$env:USERPROFILE\.m2\settings.xml`
	- or simply `~\.m2\settings.xml`

Populate the file with, or add merge with already present sections (the relevant lines have been highlighted):

```xml title="~/.m2/settings.xml" 2-8 "proxy.domain.de" "8080"
<settings>
    <proxies>
        <proxy>
            <active>true</active>
            <host>proxy.domain.de</host>
            <port>8080</port>
        </proxy>
    </proxies>
</settings>
```

and you should be good to go.

> [!info]
> This is mainly for the clojure and `deps.edn` specific stuff to work.
> If you're using [Leiningen]() as your project's dependency manager, you should make sure that the `http_proxy` and `https_proxy` environment variables are set to `http://proxy.domain.de:port` correctly (as per the [official documentation](https://codeberg.org/leiningen/leiningen/wiki/HTTP-Proxies)).

## New Project

### deps-new

If you don't already have a project setup, I'd recommend using [deps-new](https://github.com/seancorfield/deps-new), which setups a `deps.edn` project.

This should be installed as a tool, so execute

```powershell
clojure -Ttools install-latest :lib io.github.seancorfield/deps-new :as new
```

Refer to the official `README.md` section for [creating an application](https://github.com/seancorfield/deps-new?tab=readme-ov-file#create-an-application) or a [minimal scratch project](https://github.com/seancorfield/deps-new?tab=readme-ov-file#create-a-minimal-scratch-project).

> [!tip]
> If you're unfamiliar with clojure, the application template is pretty convoluted.
> Instead I'd recommend to use the _minimal scratch template_ while getting started.
> 
> Look inside `src/scratch.clj` for some hints on running your first commands.

### Leiningen

Some claim the easier way to setup a project is to use [Leiningen](https://leiningen.org).
Though most projects nowadays seem to transition to `deps.edn`, Leiningen is still viable for a lot of (especially simple projects).

It is noteworthy, though, that installation on Windows uses _Chocolatey_, so you'd need to [install it, if not already present](https://chocolatey.org/install).
If you have it available, install `lein` (using an admin PowerShell instance):

```powershell
choco install lein
```

## Visual Studio Code

Our editor will be [Visual Studio Code (or VSCode)](https://code.visualstudio.com/download).
Download and install it.

## Calva

Of course, we want a nice development environment for Clojure.
This is why we'll use [Calva](https://github.com/BetterThanTomorrow/calva).
It's a very nice batteries-included framework for Clojure development within VSCode.

To install it:
1. Open VSCode
2. Go to `File -> Preferences -> Extensions`
3. Search for `Calva`
4. Select `Calva: Clojure & ClojureScript`
5. Click on `Install`

After that, simply open your project folder inside VSCode and Calva should automagically start everything, including `clojure-lsp` as the LSP server.
