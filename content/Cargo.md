---
{"publish":true,"created":{"{ date:YYYY-MM-DD HH:mm }":null},"cssclasses":""}
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