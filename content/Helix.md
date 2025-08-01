---
{"publish":true,"created":"2025-07-30 17:44","modified":"2025-08-01T09:48:25.394+02:00","cssclasses":""}
---


For easy file editing, I’m currently trying out [Helix](https://helix-editor.com), so editing is as simple as calling `hx` on the file.

I tried out Helix some time ago but didn't find the ecosystem that appealing at that point. As the project matured some more, I found myself drawn to it again.
It aims to be a no-nonsense approach to LSP and [[Tree-sitter]] integration, coming with sane defaults and batteries included out of the box.

# Install

## [[Homebrew]]

```sh
brew install helix
```

## [[Snap]]

> [!note]
> You can add the option `--edge`, to use the _latest/edge_ channel instead of the default _latest/stable_ one.

```sh
sudo snap install helix --classic
```

## [[Flatpak]]

```sh
sudo flatpak install flathub com.helix_editor.Helix
sudo flatpak run com.helix_editor.Helix
```

## #OS/Fedora 

> [!caution]
> Unfortunately, the #OS/Fedora package seems to be out of date currently, or at least significantly lacking behind development.
> Also, the [[Rust]] install from source isn't as straightforward as some other packages, where you can simply invoke [[Cargo#binstall]].
> For these reasons, I recommend the [[Helix#Homebrew]] installation method, at least for now.

```sh
sudo dnf install helix
```

