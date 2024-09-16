# ssh-config-template

A template for modern SSH Config

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
