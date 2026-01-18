---
publish: true
created: 2025-09-19
modified: 2025-11-04T17:34:00.948+01:00
cssclasses: ""
---

Agent for communcating with a [[Tabby]] server, acting like an [[LSP]].

## Configuration

> [!info]
> For complete instructions, refer to the [helix-gpt README](https://github.com/leona/helix-gpt).

### Helix

No further configuration should be necessary for the `tabby` language server itself, as I've already included a configuration with my [[Dotfiles]].

But in general, you want to add `tabby` to the `language-servers` array of a particular language you want to support within the `languages.toml` file.

> [!tip]
> You can either add it to the global file in `~/.config/helix` file, or even better configure this on a per-project basis, by using a project-local `.helix` directory!

> [!example]
>
> ```toml title="languages.toml"
> [[language]]
> name = "python"
> language-servers = ["pyright", "pylsp", "tabby"]
> ```

You can easily view the [list of default language servers for each language](https://docs.helix-editor.com/lang-support.html) to make sure you don't omit any default ones when adding `tabby`.
