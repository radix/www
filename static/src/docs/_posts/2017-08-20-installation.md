---
layout: page
title: "Install"
subtitle: "Learn how to install Krypton and the kr command line tools."
category: start
order: 1
---

# Install Krypton + kr

## Krypton app
Go to [https://get.krypt.co](https://get.krypt.co) on your iOS or Android device and you will be redirected to the Krypton app download page on the Apple App Store (iOS) or the Google Play Store (Android).

## Kr cli
The easiest way to install `kr` on any supported machine is the following: 

```bash
$ curl https://krypt.co/kr | sh
``` 
You can check out the source or download the script locally with `curl https://krypt.co/kr > kr` and inspect it

> **Note:** The rest of this document describes the what the install script does and how to manually install `kr` from source.
<hr>

## What's inside the install script?
Below is an explanation of what the install script does on each supported platform. 

### macOS
- Download the correct homebrew bottle from GitHub and verify its hash
- Untar and install `kr`, `krd`, `kr-pkcs11.so`, and `krssh` binaries
- Backup and append to your ~/.ssh/config to point to `krd`

Equivalent command:
```bash
$ brew install kryptco/tap/kr 
```

### Debian Linux (Ubuntu, Kali)
- Install necessary software for adding new repository: `software-properties-common`, `dirmngr`, `apt-transport-https`
- Add krypt.co's binary signing key from the Ubuntu keyserver (fingerprint `C4A05888A1C4FA02E1566F859F2A29A569653940`)
- Add krypt.co's apt repository hosted at `kryptco.github.io/deb`
- run `apt-get install kr`
    
Equivalent commands:
```bash
$ sudo apt-get install software-properties-common dirmngr apt-transport-https -y 
$ sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys C4A05888A1C4FA02E1566F859F2A29A569653940 
$ sudo add-apt-repository "deb http://kryptco.github.io/deb kryptco main" # non-Kali Linux only 
$ sudo printf "deb http://kryptco.github.io/deb kryptco main" >> /etc/apt/sources.list # Kali Linux only 
$ sudo apt-get update 
$ sudo apt-get install kr -y 
```

### RPM Linux (RedHat, CentOS, Fedora)
- Add krypt.co's binary signing key from the Ubuntu keyserver (fingerprint `C4A05888A1C4FA02E1566F859F2A29A569653940`)
- Add krypt.co's yum repository hosted at `kryptco.github.io/yum`
- Run `yum install kr`

Equivalent commands:
```bash
$ sudo yum-config-manager --add-repo https://krypt.co/repo/kryptco.repo # non-fedora only 
$ sudo dnf config-manager --add-repo https://krypt.co/repo/kryptco.repo # fedora only 
$ sudo yum install kr -y 
```

## Installing from source

### macOS
*Golang & Rust are automatically installed by `brew`*
```bash
$ brew install --HEAD kryptco/tap/kr
```

### linux
Dependencies:
- [Install Go 1.8+](https://golang.org/doc/install)
- [Install Rust 1.15+ and cargo](https://www.rustup.rs/)

```bash
$ export GOPATH=${GOPATH:-$PWD}
$ go get github.com/kryptco/kr
$ cd $GOPATH/src/github.com/kryptco/kr && make install && kr restart
```

## How Kr interfaces with SSH
`kr` automatically adds the following to your `~/.ssh/config` file:

```bash
# Added by Krypton
Host * 
    PKCS11Provider /usr/local/lib/kr-pkcs11.so 
    ProxyCommand /usr/local/bin/krssh %h %p 
    IdentityFile ~/.ssh/id_krypton
    IdentityFile ~/.ssh/id_ed25519 
    IdentityFile ~/.ssh/id_rsa 
    IdentityFile ~/.ssh/id_ecdsa 
    IdentityFile ~/.ssh/id_dsa 
```

- Krypton's PKCS11Provider directs SSH to `krd` as an SSH agent by setting the `SSH_AUTH_SOCK` environment variable. It also links `~/.kr/original-agent.sock` to any already-running SSH agent. This allows krd to fallback to the original agent if necessary.

- The krssh ProxyCommand detects which host is being connected to and reads the signature returned from the remote server. These are transmitted to krd and eventually verified by the Krypton phone app.

- The IdentityFile options make sure that the Krypton public key is presented to servers when trying to log in, as well as any default-named SSH keys users already have.

# Uninstalling kr
Running `kr uninstall` will remove `kr` from your computer. In the event `kr uninstall` does not succeed, manually remove the above lines from `~/.ssh/config`. If you have also used Krypton for code sigining, remove the following lines from `~/.gitconfig`:
```
[gpg]
	program = /.../krgpg
[commit]
	gpgSign = true
```
