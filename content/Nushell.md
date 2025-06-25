---
{"publish":true,"created":"2025-05-30 15:25","cssclasses":""}
---


# Install

## #OS/Fedora 

```sh
sudo dnf install nu
```

# Additional Tools

My specific [[Dotfiles]], use various tools, so it's best to make sure they're available and set up.

> [!todo] Install
> - [[carapace-bin]]
> - [[Starship]]
> - [[zoxide]]
> - _[[eza]]_
> - _[[Zellij]]_

> [!info]
> I am personally often using [Zellij](https://zellij.dev/), despite Nushell recently implementing proper support for background jobs, as outlined in the [corresponding section of the Nushell 0.103.0 patch notes](https://www.nushell.sh/blog/2025-03-18-nushell_0_103_0.html#support-for-background-jobs-toc).
> 
> This support adds the `job spawn`, `job list`, `job kill` and `job unfreeze` (for unfreezing `Ctrl + Z` jobs) commands to Nushell.
> 
> My dotfiles don't integrate [[pueue]] (utilizing [`task.nu`](https://github.com/nushell/nu_scripts/blob/main/modules/background_task/task.nu)) anymore.

# Login Shell

If you remember correctly, we set the login shell to `bash` when creating the custom user, so you might wonder why we didn’t directly set it to `nu`.

Well, Nushell **isn’t POSIX-compliant**, and neither does it want to be. Therefore, running `nu` as a login shell might not be the absolute best experience you’ll ever have.

Instead, I include a code snippet at the bottom of my `~/.bashrc`, below the interactive check `[[ $- == *i* ]]`, which will let `nu` take over any _interactive_ shell, while scripts, etc. that expect a `POSIX` compliant shell can have their way.

```bash title="~/.bashrc" {3-7}
[[ $- == *i* ]] || return

if [[ $- == *i* && $(ps --no-header --pid $PPID --format comm) != "nu" && -z ${BASH_EXECUTION_STRING} ]]
then
  exec nu
fi
```