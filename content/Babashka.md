---
publish: true
aliases: ""
created: 2025-08-01 11:35
modified: 2026-03-16T22:44:18.725+01:00
cssclasses: ""
---


> [!quote]
> Fast native Clojure scripting runtime
>
> -- [Official Babashka website](https://babashka.org)

A better runtime for scripts than straight up [[Clojure]], with blazing fast speed.
Only implements a subset of [[Clojure]], tho.

## Why tho?

As Clojure runs on the JVM it also bears the cross of this behemoth.
The JVM is famously slow to start up and can sometimes be a bit of a resource-hog.
Especially for small scripts and tooling this seems and _is_ often times overkill.

And that's exactly where [[Babashka]] comes to the rescue!
It runs on the GraalVM, a very slim and performance-mindful runtime, which gives it superb startup time.
Babashka also has very adequate tooling and included batteries that make it perfect for all (at least my) scripting needs, especially as it often times doesn't even need a dedicated project directory/structure.
Keep in mind though, that not every Clojure language feature/library/etc. is available in Babashka and easy/native Java interoperability is not a goal (in contrast to Clojure)!

Doesn't hurt to have it installed, though!

## Install

### [[Scoop]]

```sh
scoop bucket add scoop-clojure https://github.com/littleli/scoop-clojure
scoop install babashka
```

### [[Homebrew]]

```sh
brew install borkdude/brew/babashka
```
