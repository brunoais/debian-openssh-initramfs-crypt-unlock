# Debian `openssh-initramfs`

This is a Debian package that includes OpenSSH into initramfs for the purpose of remote unlocking of an encrypted system.

A lot of tutorials on the topic 'remote unlocking of encrypted systems' describe how to set up a Dropbear SSH server instead of the more widely used OpenSSH server. In comparison with OpenSSH, Dropbear lacks in features and is not generally compatible with OpenSSH. This `openssh-initramfs` Debian package tries to solve this issue, by providing a simple way to install and configure OpenSSH in initramfs.

**Table of Contents:**

- [Build](#build)
- [Installation](#installation)
- [Configuration](#configuration)
    - [`/etc/initramfs-tools/initramfs.conf`](#etcinitramfs-toolsinitramfsconf)
    - [`/etc/openssh-initramfs/config`](#etcopenssh-initramfsconfig)
    - [`/etc/openssh-initramfs/authorized_keys`](#etcopenssh-initramfsauthorizedkeys)
    - [`/etc/openssh-initramfs/ssh_host*key*`](#etcopenssh-initramfssshhostkey)
- [Limitations](#limitations)
- [License](#license)

---

## Build

There are no dependencies required to build this Debian package. You simply have to clone/download this repository and run the build command on a Debian based distribution:

```sh
# clone the repository
git clone ttps://github.com/sichkarmg/debian-package-sshd-ssh-initramfs.git

# build
cd openssh-initramfs
dpkg-deb --build openssh-initramfs/ "openssh-initramfs_$(sed -nE 's/^Version: (.*)/\1/p' openssh-initramfs/DEBIAN/control)_all.deb"
```

Alternatively you can build the package using Docker, which doesn't require you to run a Debian system:

```sh
docker run -t --rm -v "$(pwd):/shared" -u $(id -u) debian:10 bash -c "cd /shared && dpkg-deb --build openssh-initramfs/ \"openssh-initramfs_$(sed -nE 's/^Version: (.*)/\1/p' openssh-initramfs/DEBIAN/control)_all.deb\""
```

## Installation

You can download a pre-built package from the [releases page](https://github.com/Aisbergg/debian-package-openssh-initramfs/releases) or [build your own](#build). Install the package using `dpkg`:

```sh
apt install ./openssh-initramfs_*_all.deb
```

## Configuration

| File | Required | Description |
|------|----------|-------------|
| `/etc/initramfs-tools/initramfs.conf` | yes | General configuration for mkinitramfs(8). See initramfs.conf(5). |
| `/etc/openssh-initramfs/config` | no | Configuration for `openssh-initramfs` module. |
| `/etc/openssh-initramfs/authorized_keys` | no | Extra `authorized_keys` file to be copied into the initramfs. |
| `/etc/openssh-initramfs/sshd_config` | no | The SSH daemon configuration. If this file doesn't exist, the systems configuration will be used (`/etc/ssh/sshd_config`). |
| `/etc/openssh-initramfs/ssh_host*key*` | yes | SSH host keys used by the SSH daemon. |

Whenever a file has changed, the initramfs needs to be rebuilt. Use the following command to update your initramfs:

```sh
update-initramfs -u
```

#### `/etc/initramfs-tools/initramfs.conf`

This is the general configuration for mkinitramfs. The only relevant options here are `DEVICE` and `IP`. These are used to enable and configure a network interface, so that you are able to remotely login via SSH. Read more about the options in [initramfs.conf(5)](https://manpages.debian.org/buster/initramfs-tools-core/initramfs.conf.5.en.html) and [initramfs.conf(7)](https://manpages.debian.org/buster/initramfs-tools-core/initramfs-tools.7.en.html). Example configuration:

```sh
DEVICE=enp4s0
IP=:::::enp4s0:dhcp
```

#### `/etc/openssh-initramfs/config`

The main configuration of the `openssh-initramfs` module resides in `/etc/openssh-initramfs/config`. It contains mainly three important options:

- `SSH_PORT`: The SSH port the daemon inside the initramfs should listen on. This option overwrites the configuration set in the `sshd_config`. The port should differ from the regular SSH port, so the `known_hosts` won't clash.
- `SSH_OPTIONS`: Extra options to pass to the SSH daemon. This can be used to overwrite any option of the `sshd_config`.
- `SSH_AUTHORIZED_KEYS_FROM`: In addition to define authorized keys in `/etc/openssh-initramfs/authorized_keys`, the module also allows to copy the authorized keys from users. To do so, simply add the users names to `SSH_AUTHORIZED_KEYS_FROM` space separated list. Note, that only the `root` user is available in initramfs, so any listed user has to use the `root` account instead of their regular account to log into the initramfs busybox.

#### `/etc/openssh-initramfs/authorized_keys`

You can add authorized SSH keys to the `/etc/openssh-initramfs/authorized_keys` file. Any of those keys will then be accepted when logging into the initramfs SSH server.

#### `/etc/openssh-initramfs/ssh_host*key*`

The SSH host keys are necessary for cryptographic operations and identification of the server. You should not simply copy your systems SSH host keys into the initramfs, because anything inside the initramfs will not be encrypted and might get stolen by an adversary. Therefore, it is safer to create new host keys and use those only inside the initramfs:

```sh
ssh-keygen -t ed25519 -f /etc/openssh-initramfs/ssh_host_ed25519_key -q -N ""
ssh-keygen -t ecdsa -f /etc/openssh-initramfs/ssh_host_ecdsa_key -q -N ""
ssh-keygen -t rsa -b 4096 -f /etc/openssh-initramfs/ssh_host_rsa_key -q -N ""
```

## Limitations

- PAM integration does not work yet and is therefore disabled.

## License

GPL v2 in order to facilitate potential inclusion inside Debian as official package.
