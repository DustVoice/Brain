---
publish: true
aliases: ""
created: 2025-08-01 11:31
modified: 2026-03-16T22:30:30.492+01:00
cssclasses: ""
---

A versatile and dynamically typed LISP language.
For your quick scripting needs, consider using [[Babashka]].

Also you should choose a build system. Either install and use [[clj-tools\|deps.edn]] or [[Leiningen]].

## Install

### [[Scoop]]

- The `extra` bucket is needed, as `clj-deps` depends on the `vcredist2022` (Microsoft Visual C++ 2022 Redistributable) available within it!
- `clj-deps` and many other [Clojure software packages](https://github.com/littleli/scoop-clojure#other-tools-available-in-this-bucket) (e.g., [[Babashka]]) are available from the [scoop-clojure](https://github.com/littleli/scoop-clojure) bucket/repository.

```sh
scoop bucket add extra
scoop bucket add scoop-clojure https://github.com/littleli/scoop-clojure
scoop install clj-deps
scoop update clj-deps
```

### [[Fedora]]

```sh
sudo dnf install clojure
```
