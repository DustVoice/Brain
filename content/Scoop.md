---
publish: true
created: 2025-09-19
modified: 2026-03-16T22:11:44.670+01:00
tags:
  - OS/Windows
cssclasses: ""
---

Scoop is an easy to use command-line installer for Windows.
It significantly streamlines the experience of installing and managing software (especially for development purposes)!
And the best thing? It lives in user-space as much as possible!

> [!note]
> It is always recommended to reference up-to-date documentation, particularly [Scoop's own website](https://scoop.sh) when installing!

## Install

Simply run the following command(s) from a PowerShell terminal:

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
Invoke-RestMethod -Uri https://get.scoop.sh | Invoke-Expression
```

This will automagically install scoop!

## Install a package

Generally a simple `scoop install <package>` will suffice!

It could however be that the package is not contained within the default _bucket_.
Buckets are collections of packages, organized by categories, or even supplied by external repositories.
One of the most utilized buckets would be `extras`, for example.
To add a bucket, simply do `scoop bucket add <bucket>`.

Finally, to search for a package, you can do `scoop search <package>` and it will even show you what bucket it belongs to!
