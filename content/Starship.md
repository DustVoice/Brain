---
{"publish":true,"created":"2025-05-30 15:41","modified":"2025-06-26T08:29:55.268+02:00","cssclasses":""}
---


# Install

## [[Cargo]]

```sh
cargo install starship
```

# Extensions

## [starship-jj](https://gitlab.com/lanastara_foss/starship-jj)

[[Jujutsu]] integration for Starship.

### Install

Use [[Cargo#binstall]]

```sh
cargo binstall starship-jj
```

### Use

Add the following entry to (the end of) your `starship.toml`:

```toml title="~/.config/starship.toml {1-7}
[custom.jj]
command = "prompt"
format = "$output"
ignore_timeout = true
shell = ["starship-jj", "--ignore-working-copy", "starship"]
use_stdin = false
when = true
```

Then add it using `${custom.jj}` to your prompt's `format` entry.

> [!example]-
> ```toml title="~/.config/starship.toml {3}
> format="""
> [...]
> ${custom.jj}\
> [...]
> """
> ```