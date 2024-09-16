# ssh-config-template

A template for modern SSH Config

# Table of Contents

-   [Overview](#overview)
-   [Initial Setup](#how-to-initialize-a-users-ssh-config)

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

-   the `!` prefix means _**DO NOT SHARE** between machines_
-   **_security_**: `id_*` (keys) enable attackers to login as you, wherever you have access
-   `id_ed25519` had already reached wide adoption by 2019 and is the preferred key type \
    (due to efficiency, convenience, and security)
-   `id_rsa` and `id_ecdsa` should be replaced at your earliest convenience \
    (due to inefficiency, and potential future vulnerabilities)
-   `authorized_keys` shows which keys allow login to the current user account
-   **_security_**: `config` and `config.d/*` show which systems you access, and how
-   the `known_hosts` fingerprint cache is updated each time you connect to a new host

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
