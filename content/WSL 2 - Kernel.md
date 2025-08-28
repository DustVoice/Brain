---
{"publish":true,"created":"2025-05-30 12:30","modified":"2025-05-31T00:09:29.685+02:00","cssclasses":""}
---


This topic concerns mainly WSL 2 itself, or in more precise terms, the WSL _kernel_, which is technically a separate part from the WSL _distribution_, although many people mangle the terminology.

As I will usually install a different distro than the default (Ubuntu), I won’t bother with that particular part of Microsoft’s official documentation and only install the bare kernel in this note.

# Requirements

> [!done] This is obsolete
> It seems as if you don’t longer need to manually enable the Windows Feature, as [this comment](https://devblogs.microsoft.com/commandline/install-wsl-with-a-single-command-now-available-in-windows-10-version-2004-and-higher/?commentid=5587#comment-5587) suggests.
> 
> I’ll still leave the following info intact because we all know what beautiful, reliable and stable pieces of software _Microsoft Windows_ and the _Microsoft Store_ are… `sarcasm.delete();`

You might want to enable the `Windows Subsystem for Linux` Windows feature. You can either try opening your _Windows Search_ and searching for `Turn Windows features on or off`, or you can dig your way through the _Control Panel_.

> [!note]
> For [WSLg](https://github.com/microsoft/wslg) to work properly, you need to make sure you’re running Windows 11, or the minimum required version of Windows 10 (Version 2004 or higher / Build 19041 or higher).

# Install / Update WSL

> [!info]
> Microsoft recommends installing the Distro from the Microsoft store, but also specifies the commands below in their official documentation.
> 
> In my experience, depending on your installation, or whether your Microsoft Store has shot itself in the foot yet, the installation is a little incoherent.
> 
> When in doubt, use the [official Microsoft installation documentation](https://learn.microsoft.com/en-us/windows/wsl/install).

To install WSL from a `cmd.exe`-prompt or _PowerShell_ run (with **elevated privileges**)

```ps
wsl.exe --install --no-distribution
```

or if you’ve already installed WSL in the past, run (also from an **elevated** prompt)

```ps
wsl.exe --update
```

## Just Works™?

> [!note]
> This should only be neccessary if your Windows is locked down, the Microsoft Store is disabled, or similar.

As it turns out, if you’re in a pretty restricting corporate setting, the Windows environment is somewhat … let’s say botched.
The only way I was able to install WSL successfully, was by using the manual installation method.

1. Download the [latest WSL release](https://github.com/microsoft/WSL/releases) (the `.msixbundle`)
2. Install it by running `Add-AppxPackage Microsoft.WSL_{version}_{arch}.msixbundle{:powershell}`
3. Proceed with the setting up the distribution

> [!tip]
> If `wsl --update{:ps}` doesn’t work (in addition to `wsl --install [...]{:ps}`), you might need to repeat this ordeal when you want to _update_ WSL.