---
{"publish":true,"created":"2025-06-10T13:45:36.367+02:00","modified":"2025-09-12T11:05:01.846+02:00","cssclasses":""}
---

Cargo is [[Rust]]'s superb package manager and can be used both for installing rust packages and managing a crate's dependencies.

# binstall

If you would rather not compile every package you `install` with `cargo` from source, you can utilize `cargo-binstall`.

Simply install it

```sh
cargo install cargo-binstall
```

Then replace `cargo install` with `cargo binstall` for every installation command.

Note that there has to be a binary package available.

# update

Cargo doesn't have a native update command.
Normally you would simply perform another _install_ command instead.

There is a package, though. Install it using [[Cargo#binstall]].

```sh
cargo binstall cargo-update
```

To now update all packages, simply do

```sh
cargo install-update -a
```