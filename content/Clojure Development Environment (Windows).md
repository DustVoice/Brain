---
publish: true
created: 2025-09-19
modified: 2026-03-16T23:41:25.647+01:00
tags:
  - OS/Windows
cssclasses: ""
---


> [!warning] **Deprecation Notice:** Previous Version
> The previous version of this guide was extremely manual in nature.
> It involved installing the JDK manually, together with Git, and _much more_.
>
> It turns out that actually using [[Scoop]] after all, makes this whole process way more streamlined and neat.
> In addition, using [[Clojure Development Environment (Windows)#neil]] instead of the barebones [[Clojure Development Environment (Windows)#deps-new]], is way easier for making the experience non-hair-pulling.
> Especially things like the easy addition of an _nREPL_, or `:build` alias, together with easy dependency addition and updating, makes _neil_ a really nice tool!
> Together with [[Clojure Development Environment (Windows)#Babashka]], this completely replaced my need for [[Clojure Development Environment (Windows)#Leiningen]]!

This loose guide is aimed to assist (mainly my coworkers) in setting up a Clojure development environment on Windows, preferably using Calva (within Visual Studio Code) serving as a pretty full-fledged **Clojure IDE**!

## Scoop

![[Scoop]]

## Dependencies

Now that we have setup Scoop, we need to install a couple of dependencies for both [[Clojure Development Environment (Windows)#Clojure]], as well as [[Clojure Development Environment (Windows)#Calva]] later on in the guide.

> [!info]- Source
> The following part was largely adopted from the [scoop-clojure Project's GitHub page](https://github.com/littleli/scoop-clojure).

### Git

[[Git]] _is required by Calva_ and also often times the recommended VCS (Version Control System).
Install it using scoop:

![[Git#Scoop]]

> [!hint]- Excursion - Yet another VCS
> When developing a real project, I found Git to have some short-comings in conflict resolution and branch philosophy.
> I found them to be completely mitigated by [[Jujutsu]] (jj)!
> It uses Git under the hood, so it is completely possible to share jj repositories with other (git) users, or simply use it on existing Git repositories.
> To an outsider it looks just like Git! Give it a read/try if you want.

### Java

As Clojure runs on the JVM (Java Virtual Machine), we obviously need to install Java, specifically a JDK (Java Development Kit).

> [!attention]- JDK vs. JRE (and different vendors)
> Note the difference between a **JRE** (Java _Runtime_ Environment) and a **JDK** (Java _Development_ Kit)!
>
> Many PCs probably have a JRE version installed, which will commonly be referred to an installed _Java version_.
> This will **not** allow you to **develop** in Java!
> That's what the **JDK** is for!
>
> _Though, to be exact, every JDK commonly packages a matching JRE._
>
> Also worth noting that the Java ecosystem has become somewhat ... fragmented over the years.
> Though the language is generally the same (of course for a specific Java version) there are many JDKs (from different vendors, too) out there with different licenses, and other head-spinning stuff.
> Be it the Oracle JDK, OpenJDK, Eclipse Adoptium Temurin, and many more.
> For me Temurin is one of the more hassle-free open-source non-sucky variants (on Windows).

The most pain-free approach is to install it using scoop, too!
_(You can also do a more manual approach, by [downloading it](https://adoptium.net/download), installing it and making sure `JAVA_HOME` is correctly set)._

For that you simply have to add the [`java` bucket](https://github.com/ScoopInstaller/Java) and install the (TCK certified) Java runtime and compiler:

![[Temurin#Scoop]]

## Clojure

Now we can finally install the official clojure tools!

![[Clojure#Scoop]]

### Proxy

If you are sitting behind a corporate proxy, you need to set the proxy for Clojure _(Maven to be exact)_ first!

Clojure/Maven both use the same centralized `.\m2\settings.xml` config file for that.
Under Windows the config file should be located (or not created) in your User directory (`C:\Users\<username>`)

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

### Optional Tooling

There are some (of course, optional) tools I like to use.

I use [[Clojure Development Environment (Windows)#neil]] for easily managing project dependencies, common aliases and for quick project creation, while I use [[Clojure Development Environment (Windows)#Babashka (Optional)\|Babashka]] for quick scripting that doesn't warrant a whole project structure.

#### deps-new

For a new project (and as an alternative/fallback to [[neil]]), you can simply use [deps-new](https://github.com/seancorfield/deps-new), which setups a `deps.edn` project.
It doesn't have shiny bling-bling, but works.

Especially since I had some problems with [[Clojure Development Environment (Windows)#Babashka]] (and therefore [[Clojure Development Environment (Windows)#neil]], which depends on it) working correctly behind a Proxy!
As it uses the default clojure tools, [[Clojure Development Environment (Windows)#Proxy\|the proxy setup described below]] works correctly for `deps.edn` projects created by deps-new.

deps-new should be installed as a _clojure tool_, so execute

`clojure -Ttools install-latest :lib io.github.seancorfield/deps-new :as new`

Refer to the official `README.md` section for [creating an application](https://github.com/seancorfield/deps-new?tab=readme-ov-file#create-an-application) or a [minimal scratch project](https://github.com/seancorfield/deps-new?tab=readme-ov-file#create-a-minimal-scratch-project).

> Tip
>
> If you’re unfamiliar with clojure, the application template is pretty convoluted. Instead I’d recommend to use the _minimal scratch template_ while getting started.
>
> Look inside `src/scratch.clj` for some hints on running your first commands.

#### Babashka

![[Babashka#Why tho?]]

![[Babashka#Scoop]]

#### neil

![[neil#Scoop]]

## Visual Studio Code

Our editor will be [Visual Studio Code (or VSCode)](https://code.visualstudio.com/download) and also what we will install [[Clojure Development Environment (Windows)#Calva]] in (which is actually _just_ an extension).
Simply download and install it from either the Microsoft Website using their installer, or using Scoop:

![[Visual Studio Code#Scoop]]

### Calva

Of course, we want a nice development environment for Clojure.
This is why we'll use [Calva](https://github.com/BetterThanTomorrow/calva).
It's a very nice batteries-included framework for Clojure development within VSCode.

To install it:
1. Open VSCode
2. Go to `File -> Preferences -> Extensions`
3. Search for `Calva`
4. Select `Calva: Clojure & ClojureScript`
5. Click on `Install`

After that, simply open your project folder inside VSCode and Calva should automagically start everything, including `clojure-lsp` as the LSP server, their formatter and much more.

> [!tip]
> As for keybinds and generell development workflow, please refer to the [upstream Calva documentation](https://calva.io)!
> Generally, though, you should be able to use `Ctrl + Alt + C` for some Calva-specific commands, or use the general VSCode command palette/search using `Ctrl + Shift + P`.
>
> You should try to use the "Jack-in" feature for Calva to start a REPL in you project directory, so you can easily evaluate any S-Expression, or in other non-LISPy words, statements enclosed in `()` (so ... everything) using `Alt + Enter`.
> But as I said: Use the documentation!

## Other Editors / IDEs

There are of course alternatives to Calva.
A non-exhaustive list is provided (the ones I have tried personally), though you should checkout the official [Clojure Editors list](https://clojure.org/guides/editors) for further suggestions.

### Emacs

I'm in no position to call other people savage-madmans, but running Emacs on Windows, at least natively, is something I've long given up on.
If you value your sanity, use a [[WSL 2 - Distro]] for that!

In any case, have a look at [Cider](https://github.com/clojure-emacs/cider).
Cider is one of the most feature-complete implementations of a true LISP agnostic development experience I have encountered!

### Neovim

Slightly better to run on Windows (natively) compared to [[Clojure Development Environment (Windows)#Emacs]].

There's a Cider equivalent (more or less) for Neovim called [conjure](https://github.com/Olical/conjure) (fittingly).
The good thing about it is that is also supports a bunch of other languages within and outside the LISP family, including but not limited to:
- [[Python]] (and [[Basilisp]])
- [[Fennel]] ([[Lua]] compatible Lisp)
- [[Elixir]]
- 

### Cursive

[Cursive](https://cursive-ide.com) is a Clojure(Script) IDE built on _JetBrains IntelliJ_ (a Java IDE).

It is _not free_, though there exists a [_Personal_ license available for purchase](https://cursive-ide.com/buy.html) that could satisfy your needs:

> ![quote]
> Available to individuals purchasing the licence for their own use with their own money. May not be purchased or reimbursed by companies, but may be used at work by the named user. This licence is perpetual, with free updates for a year.
> ([EULA](https://cursive-ide.com/personal-licence.html))
>
> -- [Cursive purchase page](https://cursive-ide.com/buy.html)

This means you could legally (at least as far as Cursive is concerned) use it in your job.
