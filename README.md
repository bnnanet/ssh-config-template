# ssh-config-template

A template for modern SSH Config

This was created with 3 use cases in mind:

- To help understand and initialize your `~/.ssh/` config
- To share config (NOT KEYS) among a few computers (e.g. office and laptop)
- To provide examples for common tasks (e.g. forwarding, proxying)

# Table of Contents

- [Overview](#overview)
- [Initial Setup](#how-to-initialize-a-users-ssh-config)
- [Cloning this Template](#how-to-use-this-template-repo)
- [Reasonable Defaults](#reasonable-defaults)
  - [~/.ssh/config:](#sshconfig)
  - [~/.ssh/config.d/example.sshconfig:](#sshconfigdexamplesshconfig)
- [Syntax Highlighting](#how-to-get-ssh-config-syntax-highlighting)
  - [VIM](#vim)
  - [GitHub-flavored Markdown](#github-flavored-markdown)
- [Cheat Sheet](#ssh-cli--config-cheat-sheet)
  - [Port Forward (to localhost)](#how-to-port-forward-over-ssh)
  - [SSH over TLS](#how-to-tunnel-ssh-client-over-tls)

## Overview

Here's what a properly configured `~/.ssh/` looks like:

```text
.ssh/
├── authorized_keys
├── config
├── config.d/
│   └── example.sshconfig
├── ! id_ed25519
├── ! id_ed25519.pub
└── ! known_hosts
```

Notes:

- the `!` prefix means _**DO NOT SHARE** between machines_
- **_security_**: `id_*` (keys) enable attackers to login as you, wherever you
  have access
- `id_ed25519` had already reached wide adoption by 2019 and is the preferred
  key type \
  (due to efficiency, convenience, and security)
- `id_rsa` and `id_ecdsa` should be replaced at your earliest convenience \
  (due to inefficiency, and potential future vulnerabilities)
- `authorized_keys` shows which keys allow login to the current user account
- **_security_**: `config` and `config.d/*` show which systems you access, and
  how
- the `known_hosts` fingerprint cache is updated each time you connect to a new
  host

## How to use this Template Repo

If you use this template repo to get your SSH config started (and update with
future example

1. SET TO **PRIVATE** when creating from template
   <kbd><img width="591" alt="ssh-config-template-private" src="https://github.com/user-attachments/assets/59af9b33-de38-4f52-b223-a8667ecee036"></kbd>
2. CHANGE THE **DEFAULT BRANCH** after creating \
   (e.g. to 'private' or 'john' or 'macbook')
   <kbd><img width="580" alt="ssh-config-template-branch" src="https://github.com/user-attachments/assets/d7298c0a-08fa-4d43-bd0d-e350284d181c"></kbd>

To update with new example configs:

```sh
git pull --rebase https://github.com/bnnanet/ssh-config-template.git public
```

## How to Initialize a user's SSH config

1. Create the directory structure

   ```sh
   mkdir -p ~/.ssh/config.d/
   chmod 0700 ~/.ssh/
   chmod 0700 ~/.ssh/config.d/

   touch ~/.ssh/authorized_keys
   chmod 0644 ~/.ssh/authorized_keys

   touch ~/.ssh/config
   touch ~/.ssh/config.d/example.sshconfig
   touch ~/.ssh/known_hosts
   chmod 0600 ~/.ssh/config
   chmod 0600 ~/.ssh/config.d/example.sshconfig
   chmod 0600 ~/.ssh/known_hosts
   ```

2. Create (but _NOT_ overwrite) SSH Keys

   ```sh
   if ! test -f ~/.ssh/id_ed25519; then
       # create key
       ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519 -q -N ""

       # (re)set passphrase
       ssh-keygen -p -f ~/.ssh/id_ed25519
   fi
   ```

3. Add an _EXPLICIT_ .gitignore, if saving with git \
   (ignores _EVERYTHING_, then allows individually with `!` prefix)

   ```ignore
   # ignore everything
   *

   # allow files that may reasonably be shared across machines
   # (NO KEYS!)
   !authorized_keys
   !config
   !config.d/
   !config.d/**.sshconfig

   # allow git files
   !README.md
   !.gitignore
   !.gitkeep
   ```

## Reasonable Defaults

- `~/.ssh/config` is used for global config and fallbacks
- `~/.ssh/config.d/<name>.sshconfig` is used for groups or individual host
  config

### `~/.ssh/config`:

```ssh-config
Include ~/.ssh/config.d/*.sshconfig

## Global Defaults
Host *
    User app
    Port 22
    StrictHostKeyChecking accept-new
    ServerAliveInterval 3600
    # Share sessions to the same host and keep them alive
    ControlMaster auto
    ControlPath ~/.ssh/%r@%h:%p
    ControlPersist 15m
    # Which keys to try, in order
    IdentityFile ~/.ssh/id_ed25519
    IdentityFile ~/.ssh/id_ecdsa
    IdentityFile ~/.ssh/id_rsa
```

### `~/.ssh/config.d/example.sshconfig`:

```ssh-config
Host example example.com example-XXXXXX.cloud.example.net
    # Internal IP 10.0.0.101
    Hostname example-1.ffffff.example.net
    User app
    Port 22
```

## How to get SSH Config Syntax Highlighting

### VIM

`~/.vimrc`:

```vim
autocmd BufNewFile,BufRead *.ssh,*.sshconfig setfiletype sshconfig
```

### GitHub-flavored Markdown

GitHub uses the ` ```ssh-config ` codeblock for sshconfig highlighting.

````mkdn
```ssh-config
# ssh example
Host example example.com example-XXXXXX.cloud.example.net
    # Internal IP 10.0.0.101
    Hostname example-1.ffffff.example.net
    ProxyCommand sclient --alpn ssh %h
```
````

```ssh-config
# ssh example
Host example example.com example-XXXXXX.cloud.example.net
    # Internal IP 10.0.0.101
    Hostname example-1.ffffff.example.net
    ProxyCommand sclient --alpn ssh %h
```

Follow <https://github.com/github-linguist/linguist/issues/7041> for future
updates.

## SSH CLI + Config Cheat Sheet

### How to Port Forward over SSH

This example forwards remote port 5432 on a postgres server to be able to listen
on port 54321 locally.

Create a one-off local-forward connection like this:

```sh
ssh pg-XXXXXX.cloud.example.net -L 54321:127.0.0.1:5432 -fnNT
```

Or create a reusable alias for `ssh pg-forward` using a config like this:

```ssh-config
# ssh pg
Host pg-forward
    Hostname pg-XXXXXX.cloud.example.net

    # LocalForward <local-port> <remote-network-host>:<remote-port>
    LocalForward 54321 127.0.0.1:5432

    # -f    Requests ssh to go to background just before command execution.
    ForkAfterAuthentication yes

    # -n    Documented as implied by -f, but explicit is more reliable, at least on macOS
    StdinNull yes

    # -N    Do not execute a remote command.
    SessionType none

    # -T    Disable pseudo-tty allocation.
    RequestTTY no

    # Share the connection rather than erroring when run multiple times
    ControlMaster auto
    ControlPath ~/.ssh/%r@%h:%p
    ControlPersist 15m
```

Then each time the alias is used, the config will be applied automatically:

```sh
ssh pg-forward
```

Notes:

- instead of `127.0.0.1`, you can choose another host that the ssh server has
  access to - such as forwarding web traffic from example.com:
  ```sh
  ssh webproxy-XXXXXX.cloud.example.net -L 3080:example.com:443 -fnNT
  ```

### How to tunnel SSH client over TLS

Using [sclient](https://webinstall.dev/sclient): \
(easy to remember)

```ssh-config
# sclient
ProxyCommand sclient --alpn ssh %h
```

Using `s_client`: \
(almost always already installed)

```ssh-config
# openssl s_client
ProxyCommand openssl s_client -connect %h:443 -alpn ssh -quiet 2>/dev/null
```

For how to multiplex SSH-over-TLS on the server, see
<https://github.com/mholt/caddy-l4/issues/101>.
