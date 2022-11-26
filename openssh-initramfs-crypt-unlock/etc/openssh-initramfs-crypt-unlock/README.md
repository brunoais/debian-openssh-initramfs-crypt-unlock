# Configuration files for openssh-initramfs



## Setup

### Add user

To allow a user to login using openssh (only public-private key is supported) and unlock luks containers, you add that user to `initramfs-remote-unlock` group

Example with command line:

```
usermod -aG initramfs-remote-unlock username
```

### Initramfs authentication

Users' `$HOME/.ssh/autorized_keys` are automatically copied to initramfs.

Alternatively, users may create a `$HOME/.ssh/initramfs_authorized_keys` if you want to use a different set of keys to login in initramfs


### Add configuration to sshd

    Add files to `/etc/openssh-initramfs/sshd_config.d`. Those files are added to sshd configuration in initramfs

### Override sshd configurations
    Files in `/etc/openssh-initramfs/sshd_config.d` have precedence towards files in `/etc/ssh/sshd_config.d`.
    If you want to override or simply disable a file in `/etc/ssh/sshd_config.d`, you can either:

    1. Create a file in `/etc/openssh-initramfs/sshd_config.d` with the same name as in `/etc/ssh/sshd_config.d` and leave it empty
    2. Create a file in `/etc/openssh-initramfs/sshd_config.d` which is lexicographically lower than the file in `/etc/ssh/sshd_config.d` with the same options and the target file.


## More remarks

### Login methods

Only publickey is supported. All other login methods are shown as available (on purpose) but won't work.  

Hashed passwords are not copied for initramfs at this time.
