# GPU server user manual (English)

This guide is for international students and other English-speaking users of the Nakazawa Lab GPU servers. Start with the route that applies to you. Access requires approval, registration of **your public SSH key**, and permission for the specific GPU server.

| User | Application | Connection |
| --- | --- | --- |
| Nakazawa Lab member | [Lab user application](https://forms.cloud.microsoft/r/mgFXy680J1) | `-lan` when the GPU is directly reachable; `-campus` when the campus relay is reachable |
| University user outside the lab, including seminar students | [Campus user application](https://forms.cloud.microsoft/pages/responsepage.aspx?id=Xbxum3IjmUClZWULy8MnT6rI0kF_K9lEhfaBpChXa5JUNzlWOE05NlQxWVFDNlExMVhRV09BT05YSS4u&route=shorturl) | `-campus` |

The application forms may display Japanese. Prepare the full public key and its SHA256 fingerprint before submitting. If you plan to connect from off campus, state that you will use the university's remote-VPN.

## 1. Choose your route

- **`-lan`** connects directly from your PC to an approved GPU server. Use it only where that server's SSH port is reachable.
- **`-campus`** goes through the campus relay and Quadra to an approved GPU server. Your PC must be able to reach the relay's SSH port. The administrator maintains the reverse tunnel; you do not start it.
- **From off campus**, first follow the university's [remote-VPN instructions](https://uranus.mars.kanazawa-it.ac.jp/dpc/navi_network/navi_network_5/). After connecting, use `-lan` if the GPU is directly reachable or `-campus` if the campus relay is reachable. VPN access alone does not grant a lab account or GPU permission. Contact the administrator if neither route is reachable.

The `-lan` and `-campus` names are SSH aliases for different routes to the same GPU, not different computers. The former AWS `-ssm` route is no longer used.

## 2. Create your SSH key on your own computer

These commands run on **your PC**, not on a GPU server. Use Terminal on macOS/Ubuntu or PowerShell on Windows. Check that the proposed key filename is unused:

```bash
# macOS / Ubuntu: a "No such file" result means the name is available
ls -l ~/.ssh/id_ed25519_i2lab ~/.ssh/id_ed25519_i2lab.pub
```

```powershell
# Windows PowerShell: both results should be False
Test-Path "$env:USERPROFILE\.ssh\id_ed25519_i2lab"
Test-Path "$env:USERPROFILE\.ssh\id_ed25519_i2lab.pub"
```

If either file already exists, **do not overwrite it**. Use your existing personal key or choose a new filename and update the commands and `IdentityFile` accordingly. To create a new Ed25519 key, replace `USERNAME` with your preferred FreeIPA username (or the confirmed username if you already have one):

```bash
# macOS / Ubuntu
mkdir -p ~/.ssh
chmod 700 ~/.ssh
ssh-keygen -t ed25519 -a 100 -f ~/.ssh/id_ed25519_i2lab -C "USERNAME@i2lab.test"
chmod 600 ~/.ssh/id_ed25519_i2lab
```

```powershell
# Windows PowerShell
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.ssh" | Out-Null
ssh-keygen -t ed25519 -a 100 -f "$env:USERPROFILE\.ssh\id_ed25519_i2lab" -C "USERNAME@i2lab.test"
```

Choose a strong, nonempty passphrase when prompted. It protects the private key on your PC and is separate from your FreeIPA password. The `-C` value is only a label: it does not create or select a FreeIPA account. If `ssh` is missing, install the OpenSSH client (Windows Optional Features or Ubuntu's `openssh-client` package).

Display the **public** key and its fingerprint:

```bash
# macOS / Ubuntu
cat ~/.ssh/id_ed25519_i2lab.pub
ssh-keygen -lf ~/.ssh/id_ed25519_i2lab.pub -E sha256
```

```powershell
# Windows PowerShell
Get-Content "$env:USERPROFILE\.ssh\id_ed25519_i2lab.pub"
ssh-keygen -lf "$env:USERPROFILE\.ssh\id_ed25519_i2lab.pub" -E sha256
```

Submit the entire single `ssh-ed25519 ...` line from the `.pub` file and the `SHA256:...` fingerprint. The fingerprint identifies the key; it does not replace the public key. Also provide your preferred username, identity/affiliation, purpose and period of use, requested GPU and storage, and connection route. For campus access, the administrator must register your key for both the `campus-relay` account on the relay and your own FreeIPA account on Quadra/GPU. The relay username is shared, but each user authenticates with their own key.

**Never submit or copy the private key** (the file without `.pub`), its passphrase, or your password to a form, server, repository, issue, or chat. If you see `-----BEGIN OPENSSH PRIVATE KEY-----`, you opened the wrong file.

## 3. Wait for approval, then configure SSH

The administrator will confirm your FreeIPA username, registered public-key fingerprint, approved GPU and storage, and server host-key fingerprints. Campus users also need confirmation of relay registration and forwarding permission. Generating a key or submitting a form does not itself enable access.

Copy the relevant `Host` blocks from the **canonical configuration examples** into your own SSH config. Do not edit the example file and expect it to take effect.

- [Direct `-lan` configuration](../../config/ssh_config.example)
- [Campus `-campus` configuration](../../config/ssh_config.campus.example): include `campus-jump`, `quadra-campus`, and only your approved GPU hosts

Save your config as `~/.ssh/config` on macOS/Ubuntu or `%USERPROFILE%\.ssh\config` (no extension) on Windows. Back up an existing config, update any matching `Host` blocks, and keep unrelated entries. Replace `__SSH_USER__` with the **confirmed** FreeIPA username and `__IDENTITY_FILE__` with the private-key path (without `.pub`). Your PC login name may differ from the FreeIPA username. On Windows, a key path can be written as `"C:/Users/YOUR_WINDOWS_NAME/.ssh/id_ed25519_i2lab"`.

For macOS/Ubuntu, follow the example's socket-directory instructions and set `chmod 600 ~/.ssh/config`. For the standard Windows OpenSSH client, remove the `ControlMaster`, `ControlPath`, and `ControlPersist` lines from copied blocks. Do not copy the private key to the relay or servers or enable SSH agent forwarding. The campus relay uses `User campus-relay`; Quadra and GPU blocks use your confirmed FreeIPA username. The campus config's `127.0.0.1:2222` is the relay-side tunnel endpoint; leave it as provided.

## 4. Verify the connection

Run these on **your PC**, replacing RTX5090 with a GPU you were approved to use. `ssh -G` only checks the settings loaded on your PC; it does not test access.

```bash
ssh -G rtx5090-lan
ssh rtx5090-lan 'hostname -f; whoami; id; nvidia-smi'
```

For campus access, check each stage in order:

```bash
ssh -G campus-jump
ssh -G quadra-campus
ssh -G rtx5090-campus
ssh quadra-campus 'hostname -f; whoami'
ssh rtx5090-campus 'hostname -f; whoami; id; nvidia-smi'
```

Run only the commands for your route. For `-campus`, `ssh -G` should show `campus-relay` for `campus-jump`, your FreeIPA username for Quadra/GPU, your private-key path, and `quadra-campus` as the GPU's `proxyjump`. Do **not** test the relay with `ssh campus-jump 'whoami'`: it permits forwarding, not shell commands.

On the first real connection, compare each displayed **server host-key fingerprint** with the value supplied by the administrator before accepting it. This differs from your public key's fingerprint. If it does not match or SSH warns of a changed host key, stop and contact the administrator; do not remove the old record without checking. For RTX5090, expect `rtx5090.i2lab.test`, your confirmed username and groups, and GPU information. Quadra should report `quadra.i2lab.test`. `nvidia-smi` confirms that the host sees the GPU; it does not validate your research software.

If you use VS Code Remote-SSH, first verify ordinary SSH, then choose your approved `-lan` or `-campus` alias and check `hostname -f` and `whoami` in the remote terminal.

## 5. Storage and Docker

Use only the storage path and group specified in your approval. GPU servers expose approved NFS storage under `/mnt/nfs-shared/...`; an approved `labusers` user may have `/mnt/nfs-shared/labusers`, while approved `go2poc` users may have `/mnt/nfs-shared/go2poc`. Local files on different GPU servers are not automatically synchronized. For a small read/write check on an **approved** directory, log into the GPU and run:

```bash
cd /mnt/nfs-shared/labusers  # change this to your approved path
check_file=$(mktemp ./campus-check.XXXXXXXX) && {
    printf 'campus access check\n' > "$check_file"
    cat "$check_file"
    stat -c '%U:%G %a %n' "$check_file"
    rm -- "$check_file"
}
```

If you have Docker permission, run `docker version` and `docker ps` **on the GPU server**. `docker version` should show client and server information; `docker ps` may legitimately be empty. Docker permission is separate from SSH permission and effectively grants root-level control of the shared Docker environment. Do not modify other users' containers, images, volumes, networks, or monitoring services. Agree on the image, GPU, container name, ports, run time, and persistent storage with your project supervisor. Mount approved host/NFS storage for results; files kept only inside a deleted container will be lost. Check GPU access inside the actual container and framework before starting research. Do not use `sudo`, change Docker socket permissions, or run `docker system prune` to work around an error. See the [Japanese Docker guide](../docker-user-guide.md) for further operational detail.

## 6. Troubleshooting and contact

| Symptom | What to check |
| --- | --- |
| `Could not resolve hostname ...-campus` | SSH config filename/location and `Host` alias |
| Timeout to the relay or GPU | Network/VPN connection, route reachability, and time of failure |
| Password prompt or `Permission denied (publickey)` | Confirmed username, private-key path, and public-key registration on the host that rejected you |
| `127.0.0.1:2222` refused / `administratively prohibited` | Ask the administrator to check the maintained tunnel or forwarding permission |
| Quadra works, GPU fails | Approved GPU, host alias, and GPU-side permission |
| Docker permission error | Report it; the administrator checks account groups and the service |

Send the administrator the connection alias, time, source network, OS, error text, and `hostname -f`/`whoami` results where available. Do not send passwords, passphrases, private keys, or research data. If your PC or private key is lost or compromised, contact the administrator immediately so the registered key can be revoked (on both the relay and FreeIPA for campus users).

**Contact:** Minoru Nakazawa, [nakazawa@infor.kanazawa-it.ac.jp](mailto:nakazawa@infor.kanazawa-it.ac.jp).
