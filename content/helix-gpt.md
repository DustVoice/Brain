---
{"publish":true,"created":"2025-09-19","modified":"2025-10-16T22:41:42.423+02:00","cssclasses":""}
---


> [!quote]
> Code assistant language server for [[Helix]] with support for Copilot/OpenAI/Codeium.

## Installation

### Without [[Bun]]

```sh
wget https://github.com/leona/helix-gpt/releases/download/0.34/helix-gpt-0.34-x86_64-linux.tar.gz \
-O /tmp/helix-gpt.tar.gz \
&& tar -zxvf /tmp/helix-gpt.tar.gz \
&& sudo mv helix-gpt-0.34-x86_64-linux /usr/bin/helix-gpt \
&& sudo chmod +x /usr/bin/helix-gpt
```

### With [[Bun]]

```sh
wget https://github.com/leona/helix-gpt/releases/download/0.34/helix-gpt-0.34.js -O /usr/bin/helix-gpt
```

## Configuration

> [!info]
> For complete instructions, refer to the [project's README on GitHub](https://github.com/leona/helix-gpt).

### [[Copilot]]

To obtain the copilot token, execute the following command after [[helix-gpt#Installation]].

```sh
helix-gpt --authCopilot
```

It will ask you to open [GitHub's device registration page](https://github.com/login/device)
and input the 8 character code.
After that it should automatically display the obtained token.

Simply store that in the `COPILOT_API_KEY` environment variable, in my case, as I'm using [[fish]], it's as easy as

   ```fish "TOKEN"
   set -Ux COPILOT_API_KEY ghu_TOKEN
   ```

   > [!todo] Replace
> - `ghu_TOKEN`: Your obtained Copilot API token

Set the `HANDLER` environment variable to `copilot` (I'm using [[fish]] again)

   ```fish
   set -Ux HANDLER copilot
   ```

### Helix

No further configuration should be necessary for the `gpt` language server itself, as I've already included a configuration with my [[Dotfiles]].

But in general, you want to add `gpt` to the `language-servers` array of a particular language you want to support within the `languages.toml` file.

> [!tip]
> You can either add it to the global file in `~/.config/helix` file, or even better configure this on a per-project basis, by using a project-local `.helix` directory!

> [!example]
>
> ```toml title="languages.toml"
> [[language]]
> name = "python"
> language-servers = ["pyright", "pylsp", "gpt"]
> ```

You can easily view the [list of default language servers for each language](https://docs.helix-editor.com/lang-support.html) to make sure you don't omit any default ones when adding `gpt`.
