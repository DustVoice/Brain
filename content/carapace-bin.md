---
{"publish":true,"created":"2025-05-30 15:40","modified":"2025-05-31T00:09:29.803+02:00","cssclasses":""}
---


# Install

## #OS/Fedora 

Add a repo file under `/etc/yum.repos.d/fury.repo`

```config title="/etc/yum.repos.d/fury.repo"
[fury]
name=Gemfury Private Repo
baseurl=https://yum.fury.io/rsteube/
enabled=1
gpgcheck=0
```

Update

```sh
sudo dnf update
```

Then install the package

```sh
sudo dnf install carapace-bin
```