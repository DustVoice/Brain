---
share: true
created: 2025-05-30 15:25
tags: 
---

# Install

## #OS/Fedora 

```sh
sudo dnf install nu
```

# Additional Tools

My specific [[Dotfiles|Dotfiles]], use various tools, so it's best to make sure they're available and set up.

> [!todo] Install
> - [[carapace-bin|carapace-bin]]
> - [[Starship|Starship]]
> - [[zoxide|zoxide]]
> - _[[eza|eza]]_
> - _[[Zellij|Zellij]]_

> [!info]
> I am personally often using [Zellij](https://zellij.dev/), despite Nushell recently implementing proper support for background jobs, as outlined in the [corresponding section of the Nushell 0.103.0 patch notes](https://www.nushell.sh/blog/2025-03-18-nushell_0_103_0.html#support-for-background-jobs-toc).
> 
> This support adds the `job spawn`, `job list`, `job kill` and `job unfreeze` (for unfreezing `Ctrl + Z` jobs) commands to Nushell.
> 
> My dotfiles also still integrate [[pueue|pueue]], utilizing [`task.nu`](https://github.com/nushell/nu_scripts/blob/main/modules/background_task/task.nu).
