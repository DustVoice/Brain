---
share: true
created: 2025-05-30 15:37
tags: 
---

I use Rust btw.

# Install

## #OS/Fedora 

> [!NOTE]
> Also consider the [official documentation](https://developer.fedoraproject.org/tech/languages/rust/rust-installation.html).

### By using `rustup`

This is the [officially recommended variant](https://www.rust-lang.org/learn/get-started).

```sh
sudo dnf install rustup
rustup-init
```

### By using `rust`

```sh
sudo dnf install rust cargo clippy rust-src rustfmt
```

# Cargo

Cargo is rust's superb package manager and can be used both for installing rust packages and managing a crate's dependencies.

> [!tip] Cargo's **b**install
> If you would rather not compile every package you `install` with `cargo` from source, you can utilize `cargo-binstall`.
> 
> Simply install it
> 
> ```sh
> cargo install cargo-binstall
> ```
> 
> Then replace `cargo install` with `cargo binstall` for every installation command.
> 
> Note that there has to be a binary package available.
