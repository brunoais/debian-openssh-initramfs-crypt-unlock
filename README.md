# Debian `openssh-initramfs-crypt-unlock`

This is a Debian package that includes OpenSSH into initramfs for the purpose of remote unlocking of an encrypted system.

A lot of tutorials on the topic 'remote unlocking of encrypted systems' describe how to set up a Dropbear SSH server instead of the more widely used OpenSSH server. In comparison with OpenSSH, Dropbear lacks in features and is not generally compatible with OpenSSH. This `openssh-initramfs-crypt-unlock` Debian package tries to solve this issue, by providing a simple way to install and configure OpenSSH in initramfs.

## Acceptable Issues and PR

This is a personal project I share with you. Given such:

* Issues with bugs are acceptd
* Issues with feature requests will probably be left undone
* Issues are not support forums. Use discussion instead

#### Differences vs upstream Debian `openssh-initramfs`

1. Defining which users can login remotely to initramfs is done by adding them to a group (`openssh-initramfs-crypt-unlock`) instead of a configuration file.
2. Users can, optionally, create a file `$HOME/.ssh/initramfs_authorized_keys` for alternative keys for initramfs login
3. There's no central `authorized_keys` file
4. `ssh_host*key*` is optional (although, if doesn't exist, a warning is shown)
5. Initramfs sshd is configured using files in a `.d` directory and syntax is checked while running initramfs instead of a variable in a `config` file
   1. Individual files in `/etc/openssh-initramfs-crypt-unlock/sshd_config.d` with the same name as the ones in `/etc/ssh/sshd_config.d` take precedence

**Table of Contents:**

- [Debian `openssh-initramfs-crypt-unlock`](#debian-openssh-initramfs-crypt-unlock)
  - [Acceptable Issues and PR](#acceptable-issues-and-pr)
      - [Differences vs upstream Debian `openssh-initramfs`](#differences-vs-upstream-debian-openssh-initramfs)
  - [Installation](#installation)
  - [Build](#build)
  - [Configuration](#configuration)
    - [Quick settings](#quick-settings)
    - [In more detail](#in-more-detail)
      - [`/etc/initramfs-tools/initramfs.conf`](#etcinitramfs-toolsinitramfsconf)
      - [`/etc/openssh-initramfs-crypt-unlock/config.d`](#etcopenssh-initramfs-crypt-unlockconfigd)
      - [`/etc/openssh-initramfs-crypt-unlock/sshd_config.d`](#etcopenssh-initramfs-crypt-unlocksshd_configd)
      - [`/etc/openssh-initramfs-crypt-unlock/host_keys.d/`](#etcopenssh-initramfs-crypt-unlockhost_keysd)
  - [Limitations](#limitations)
  - [License](#license)

---


## Installation

You can download a pre-built package from the [releases page](https://github.com/brunoais/debian-package-openssh-initramfs-crypt-unlock/releases) or [build your own](#build). Install the package using `dpkg`:

```sh
apt install ./openssh-initramfs-crypt-unlock_*_all.deb
```

## Build

There are no dependencies required to build this Debian package. You simply have to clone/download this repository and run the build command on a Debian based distribution:

```sh
# clone the repository
git clone https://github.com/brunoais/debian-openssh-initramfs-crypt-unlock.git

# build
cd openssh-initramfs-crypt-unlock
dpkg-deb --build openssh-initramfs-crypt-unlock/ "openssh-initramfs-crypt-unlock_$(sed -nE 's/^Version: (.*)/\1/p' openssh-initramfs-crypt-unlock/DEBIAN/control)_all.deb"
```

Alternatively you can build the package using Docker, which doesn't require you to run a Debian system:

```sh
docker run -t --rm -v "$(pwd):/shared" -u $(id -u) debian:10 bash -c "cd /shared && dpkg-deb --build openssh-initramfs-crypt-unlock-crypt-unlock/ \"openssh-initramfs-crypt-unlock-crypt-unlock_$(sed -nE 's/^Version: (.*)/\1/p' openssh-initramfs-crypt-unlock-crypt-unlock/DEBIAN/control)_all.deb\""
```

## Configuration

### Quick settings

These settings are incomplete and insecure. However, they will get you started so you can grab a feel.

These steps assume you are already asked for password at boot by cryptsetup.

After installing:
```sh
# add yourself to the group:
sudo usermod -aG openssh-initramfs-crypt-unlock "$USER"
# Remake initramfs:
sudo update-initramfs -u
```
You can reboot and try it out now.

### In more detail

| File | Required | Description |
|------|----------|-------------|
| `/etc/initramfs-tools/initramfs.conf` | yes | General configuration for mkinitramfs(8). See initramfs.conf(5). |
| `/etc/openssh-initramfs-crypt-unlock/config.d` | yes | Configuration for `openssh-initramfs-crypt-unlock` module. |
| `/etc/openssh-initramfs-crypt-unlock/sshd_config` | no | if defined, overrides `/etc/ssh/sshd_config`. An `Include` is forcefully added to it so it processes the settings in `/etc/openssh-initramfs-crypt-unlock/sshd_config.d` |
| `/etc/openssh-initramfs-crypt-unlock/sshd_config.d` | yes | One is created for you with the default configuration file inside. |
| `/etc/openssh-initramfs-crypt-unlock/ssh_host*key*` | No | SSH host keys used by the SSH daemon. If none are found, the installation process will make new ones for you |

Whenever a file has changed, the initramfs needs to be rebuilt. Use the following command to update your initramfs:

```sh
update-initramfs -u
```

#### `/etc/initramfs-tools/initramfs.conf`

This is the general configuration for mkinitramfs. The only relevant options here are `DEVICE` and `IP`. These are used to enable and configure a network interface, so that you are able to remotely login via SSH. Read more about the options in [initramfs.conf(5)](https://manpages.debian.org/buster/initramfs-tools-core/initramfs.conf.5.en.html) and [initramfs.conf(7)](https://manpages.debian.org/buster/initramfs-tools-core/initramfs-tools.7.en.html). 

Sometimes, those settings aren't required (E.g. if `network-manager` is already installed and you are using DHCP)
Example configuration in case it's needed:

```sh
DEVICE=enp4s0
IP=:::::enp4s0:dhcp
```

#### `/etc/openssh-initramfs-crypt-unlock/config.d`

The main configuration of the `openssh-initramfs-crypt-unlock` itself resides in `/etc/openssh-initramfs-crypt-unlock/config.d`. Currently, it supports:

- `COPY_SSHD_CONFIG`: Set this option to `n` if you want to completely separate sshd settings at host and at initramfs
- `IFDOWN`: Set which network interfaces to shutdown (clean) before the main OS init is started. By default, all are terminated.

#### `/etc/openssh-initramfs-crypt-unlock/sshd_config.d`

The main configuration of `sshd` managed by `openssh-initramfs-crypt-unlock`. It supports all settings `sshd` supports.

When generating initramfs, the files initially copied from `/etc/ssh/sshd_config.d` are overritten with files in this directory with the same name.

`openssh-initramfs-crypt-unlock` comes with a file that sets a few highly recommended settings

- `Port`: This option overwrites the default port 22 of `sshd`. The port should differ from the regular SSH port, so the `known_hosts` won't clash. Additionally, allows better management of what key should be sent, in case of separate known_hosts files by users
- `MaxAuthTries`: The least tries the adversary gets until he's disconnected, the better. That's what this allows. By default, it's set to 1.
- `ForceCommand`: In order to avoid doing more than just unlocking and also to facilitate sending keyfiles, `ForceCommand` is, by default, set to `/usr/bin/cryptroot-unlock`
- `AllowAgentForwarding No` + `AllowTcpForwarding no`: Initramfs is not for forwarding
- `usePAM no`: There's no PAM in initramfs


#### `/etc/openssh-initramfs-crypt-unlock/host_keys.d/`

The SSH host keys are necessary for cryptographic operations and identification of the server. You should not simply copy your systems SSH host keys into the initramfs, because anything inside the initramfs will not be encrypted and might get stolen by an adversary. Therefore, it is safer to create new host keys and use those only inside the initramfs.

Given such, upon configuration step, this command is run for you:

```sh
ssh-keygen -t ed25519 -f /etc/openssh-initramfs-crypt-unlock/host_keys.d/ssh_host_ed25519_key -q -N ""
ssh-keygen -t dsa -f /etc/openssh-initramfs-crypt-unlock/host_keys.d/ssh_host_dsa_key -q -N ""
ssh-keygen -t ecdsa -f /etc/openssh-initramfs-crypt-unlock/host_keys.d/ssh_host_ecdsa_key -q -N ""
ssh-keygen -t rsa -b 4096 -f /etc/openssh-initramfs-crypt-unlock/host_keys.d/ssh_host_rsa_key -q -N ""
```
You may also run that command yourself if you believe those keys have been compromised (don't forget to warn all clients if you do so)

## Limitations

- PAM integration does not work.

## License

GPL v2 in order to facilitate potential inclusion inside Debian as official package.
