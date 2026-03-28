---
publish: true
created: 2025-09-19
---


> [!quote]
> A CLI to add common aliases and features to `deps.edn`-based projects.
>
> -- [Project's GitHub page](https://github.com/babashka/neil)

Neil is your best friend when it comes to managing aliases and dependencies of your Clojure projects, or even creating them!

## Install

### [[Scoop]]

> [!note]
> You have to have [[Babashka]] installed and available!

```sh
scoop bucket add scoop-clojure https://github.com/littleli/scoop-clojure
scoop install neil
```

## Update

Either update a specific package

```sh "<package>"
scoop update <package>
```

or every installed one

```sh
scoop update -a
```
