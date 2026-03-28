---
publish: true
created: 2025-05-02 14:03
---


Make sure your SSH key's public key is present in the `.ssh/authorized_keys` file of the user you want to use them for:
- `/home/user/.ssh/authorized_keys` for a normal user
- `/root/.ssh/authorized_keys` for root

Furthermore, make sure the directory and files have the correct permissions.

```sh
chmod 700 .ssh
chmod 600 .ssh/authorized_keys
```

Test your access to the server using your SSH key (_identity file_) as a parameter to `-i`

```sh /your_private_key/ /username/ /host/
ssh -i ~/.ssh/your_private_key username@host
```

> [!danger]
> **Make 100% sure you have configured and tested your access to the server using SSH keys sufficiently before proceeding!**
>
> You can most definitely lose access to your server.
> Especially if you don't have physical access to it.
> It might be recoverable with the help of your hosting provider, but **it might also be unrecoverable**.

Create a new file in the `/etc/ssh/sshd_config.d/` directory, to disable any access through passwords.

```conf title="/etc/ssh/sshd_config.d/90-disable_passwords.conf
PasswordAuthentication no
PermitRootLogin prohibit-password
```

Then restart `sshd`

```sh
sudo systemctl restart sshd
```

or even better, restart your system

```sh
sudo systemctl reboot
```
