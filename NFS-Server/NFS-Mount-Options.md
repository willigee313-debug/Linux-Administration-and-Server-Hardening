# NFS Mount Options

## Overview

Where [NFS-Exports-Configuration](NFS-Exports-Configuration.md) controls what the *server* offers, **mount options** control how the *client* consumes an export: which protocol version and transport it negotiates, how it caches, how it recovers from a server outage, how long it retries, and which security flavor it requests. Correct mount options are the difference between an NFS mount that survives a server reboot gracefully and one that hangs every process touching it.

This note is a reference for the mount options used with `mount -t nfs`, in `/etc/fstab`, and in systemd `.mount`/automount units, with the same options applying to Debian/Ubuntu and RHEL-family clients. For the end-to-end client setup workflow, see [NFS-Client-Setup-and-Mounting](NFS-Client-Setup-and-Mounting.md).

> [!NOTE]
> Client mount options and server export options are independent negotiations. A client may *request* `rw`, but if the export is `ro` the mount is read-only. Likewise `sec=krb5` must be offered by the server (see [NFS-with-Kerberos](NFS-with-Kerberos.md)) before a client can use it.

## Concepts

### How an NFS mount is expressed

The same option set can be supplied three ways:

```bash
# 1. Ad-hoc on the command line
sudo mount -t nfs -o vers=4.2,rw,hard,timeo=600 192.168.1.10:/srv/nfs/data /mnt/data

# 2. Persistently in /etc/fstab (mounted at boot / on demand)
#    192.168.1.10:/srv/nfs/data  /mnt/data  nfs  vers=4.2,rw,hard,timeo=600  0  0

# 3. As a systemd .mount / .automount unit (see the Examples section)
```

### Protocol version and transport

| Option | Meaning |
| --- | --- |
| `vers=3` / `nfsvers=3` | Force NFSv3 (needs `rpcbind`, `mountd`, `lockd`) |
| `vers=4` / `vers=4.0` | Force NFSv4.0 |
| `vers=4.1` | NFSv4.1 (adds sessions, pNFS) |
| `vers=4.2` | NFSv4.2 (server-side copy, sparse files, `security_label`) |
| `proto=tcp` | Use TCP (default and required for NFSv4) |
| `proto=udp` | Use UDP (NFSv3 only; discouraged) |
| `port=N` | Non-default server port |

> [!TIP]
> Prefer the newest version both ends support — `vers=4.2` on a modern stack. NFSv4 uses a single TCP port (2049), which is far easier to firewall than NFSv3's RPC port sprawl. See [NFS-Security-Hardening](NFS-Security-Hardening.md) for pinning versions.

### hard vs soft — the most consequential choice

| Option | Behavior on server unavailability | Data safety |
| --- | --- | --- |
| `hard` (default) | Retry indefinitely; I/O blocks until the server returns | Safe — no silent data loss |
| `soft` | Give up after `retrans` × `timeo` and return an error (`EIO`) | Risky — can corrupt data on write |

> [!WARNING]
> Use `hard` for anything read-write. With `soft`, a transient network blip can cause a write to fail mid-operation and silently corrupt a file. If unresponsive mounts causing hangs are the concern, combine `hard` with `intr` (older kernels) or rely on modern kernels where fatal signals can interrupt a `hard` mount, and prefer `soft` only for read-only, non-critical data.

## Configuration

### Reliability and timing options

| Option | Default | Purpose |
| --- | --- | --- |
| `hard` | yes | Retry forever (recommended) |
| `soft` | no | Fail after retries |
| `timeo=N` | 600 (TCP, deciseconds) | Retransmit timeout; `600` = 60 seconds |
| `retrans=N` | 3 | Retransmission attempts before a `soft` timeout / warning |
| `intr` / `nointr` | deprecated | Legacy interrupt of `hard` mounts (no-op on modern kernels) |
| `bg` / `fg` | `fg` | Retry mount in background if the first attempt fails |
| `retry=N` | 2 (fg) / 10000 (bg) | Minutes to keep retrying the mount itself |

### Caching and performance options

| Option | Default | Purpose |
| --- | --- | --- |
| `rsize=N` | negotiated | Read block size in bytes (e.g. 1048576) |
| `wsize=N` | negotiated | Write block size in bytes |
| `ac` / `noac` | `ac` | Enable/disable attribute caching (`noac` = strict consistency, slow) |
| `actimeo=N` | — | Set all attribute-cache timeouts to N seconds |
| `acregmin`/`acregmax` | 3 / 60 | Min/max attribute cache time for files |
| `acdirmin`/`acdirmax` | 30 / 60 | Min/max attribute cache time for directories |
| `lookupcache=all\|none\|pos\|positive` | all | Control dentry caching |
| `nocto` | off | Skip close-to-open cache revalidation (faster, weaker consistency) |

### Access, security, and mapping options

| Option | Default | Purpose |
| --- | --- | --- |
| `rw` / `ro` | server-dependent | Request read-write / read-only |
| `sec=sys` | sys | AUTH_SYS — trust client UID/GID (no real auth) |
| `sec=krb5` | — | Kerberos authentication only |
| `sec=krb5i` | — | Kerberos auth + integrity (checksums) |
| `sec=krb5p` | — | Kerberos auth + integrity + privacy (encryption) |
| `nosuid` | off | Ignore setuid/setgid bits on the mount (**recommended**) |
| `nodev` | off | Ignore device files on the mount (**recommended**) |
| `noexec` | off | Forbid execution of binaries from the mount |
| `nolock` | off | Disable NLM file locking (NFSv3 only; use sparingly) |
| `_netdev` | — | Mark as network-dependent so it mounts after the network is up |

## Commands

```bash
# Mount with an explicit, production-safe option set (NFSv4.2)
sudo mount -t nfs -o vers=4.2,rw,hard,timeo=600,retrans=3,nosuid,nodev,_netdev \
  192.168.1.10:/srv/nfs/data /mnt/data

# Show every NFS mount and its NEGOTIATED options (what the kernel actually used)
mount -t nfs4,nfs
nfsstat -m

# Remount to change options without unmounting fully (e.g. flip to read-only)
sudo mount -o remount,ro /mnt/data

# Cleanly unmount; if the server is gone and it hangs, force/lazy unmount
sudo umount /mnt/data
sudo umount -f /mnt/data     # force
sudo umount -l /mnt/data     # lazy (detach now, clean up when free)
```

> [!NOTE]
> `nfsstat -m` is the authoritative way to see the *effective* options — including values negotiated by the kernel (`rsize`, `wsize`, `vers`) that you did not set explicitly.

## Examples

### Example 1 — Persistent read-write mount in `/etc/fstab`

```conf
# /etc/fstab
# <server>:<export>        <mountpoint>   <type>  <options>                                   <dump> <pass>
192.168.1.10:/srv/nfs/data /mnt/data      nfs     vers=4.2,rw,hard,timeo=600,nosuid,nodev,_netdev  0      0
```

```bash
# Apply without rebooting (mounts everything in fstab not yet mounted)
sudo mount -a
findmnt /mnt/data
```

### Example 2 — Hardened read-only share

```conf
# /etc/fstab  -  read-only, no setuid, no devices, no execution
192.168.1.10:/srv/nfs/iso /mnt/iso nfs vers=4.2,ro,hard,nosuid,nodev,noexec,_netdev 0 0
```

### Example 3 — On-demand mount with systemd automount

Systemd automount mounts the share only when the path is first accessed and unmounts it after an idle period — ideal for shares that are not always needed.

```systemd
# /etc/systemd/system/mnt-data.mount
[Unit]
Description=NFS mount for /mnt/data
After=network-online.target
Wants=network-online.target

[Mount]
What=192.168.1.10:/srv/nfs/data
Where=/mnt/data
Type=nfs
Options=vers=4.2,rw,hard,timeo=600,nosuid,nodev,_netdev

[Install]
WantedBy=multi-user.target
```

```systemd
# /etc/systemd/system/mnt-data.automount
[Unit]
Description=Automount NFS /mnt/data

[Automount]
Where=/mnt/data
TimeoutIdleSec=600

[Install]
WantedBy=multi-user.target
```

```bash
# Enable the automount unit (NOT the .mount unit) so it mounts on first access
sudo systemctl daemon-reload
sudo systemctl enable --now mnt-data.automount
```

> [!NOTE]
> **📸 Screenshot**
> _Capture: Terminal output of nfsstat -m showing the mount point, server export, and negotiated flags including vers=4.2, rsize=1048576, wsize=1048576, hard, and timeo=600_

## Best Practices

- Default to `vers=4.2` (or the newest both ends support), `proto=tcp`, and `hard`.
- Always add `nosuid` and `nodev` to NFS mounts; add `noexec` where code should never run from the share.
- Use `_netdev` (fstab) or `After=network-online.target` (systemd) so mounts wait for the network — this prevents boot hangs.
- Prefer systemd `.automount` for shares that are used intermittently, so a downed server never blocks boot.
- Do not tune `rsize`/`wsize` manually unless benchmarking shows a benefit; the negotiated defaults are usually optimal.
- Avoid `soft` for read-write data; avoid `nolock` unless a specific application requires it.
- Verify effective options with `nfsstat -m` after mounting — do not assume your requested options were all honored.

## Security Considerations

- `sec=sys` (the default) provides **no authentication** — the server trusts whatever UID the client presents. On untrusted networks use `sec=krb5p` for encrypted, authenticated access; see [NFS-with-Kerberos](NFS-with-Kerberos.md).
- `nosuid` blocks a malicious or compromised server from serving a setuid-root binary that a client user could execute to escalate — a real cross-host privilege-escalation path. Treat `nosuid,nodev` as mandatory on NFS mounts.
- `noexec` further limits an NFS export from being used to stage and run tooling.
- Traffic under `sec=sys`/`sec=krb5`/`sec=krb5i` is **not encrypted**; file contents cross the wire in cleartext. Use `sec=krb5p` or tunnel over a VPN/WireGuard segment for confidentiality.
- These client-side controls complement server hardening in [NFS-Security-Hardening](NFS-Security-Hardening.md); neither side alone is sufficient.

## Troubleshooting

| Symptom | Likely cause | Fix |
| --- | --- | --- |
| Processes hang uninterruptibly on the mount | `hard` mount + server unreachable (expected behavior) | Restore the server/network, or `umount -f` / `umount -l` |
| `mount.nfs: Connection timed out` | Firewall blocking 2049 (or 111/mountd on v3) | Open ports; verify `proto`/`vers`; see [NFS-Security-Hardening](NFS-Security-Hardening.md) |
| `mount.nfs: access denied by server` | Export doesn't list this client | Fix `/etc/exports` server-side (see [NFS-Exports-Configuration](NFS-Exports-Configuration.md)) |
| `mount.nfs: requested NFS version or transport not supported` | Version mismatch | Set a `vers=` both sides support |
| Stale file handle (`ESTALE`) | Export was recreated / file removed server-side | Remount the share |
| Slow throughput | Poor `rsize`/`wsize` negotiation or `sync` server export | Check `nfsstat -m`; review server `async`/`sync` trade-off |
| Boot hangs waiting for NFS | Missing `_netdev` / no `network-online.target` ordering | Add `_netdev` or use systemd automount |

```bash
# Inspect negotiated options and per-mount statistics
nfsstat -m
findmnt -t nfs4,nfs

# Verbose mount attempt to see the negotiation
sudo mount -v -t nfs -o vers=4.2 192.168.1.10:/srv/nfs/data /mnt/data
```

## References

- `man 5 nfs` — the authoritative list of NFS mount options
- `man 8 mount.nfs` — the NFS mount helper
- `man 5 fstab` / `man 5 systemd.mount` / `man 5 systemd.automount`
- Red Hat Enterprise Linux — *Configuring an NFS client*
- Ubuntu Server Guide — *Network File System (NFS)*
- Linux NFS FAQ — <https://linux-nfs.org/>

## Related

- [NFS-Client-Setup-and-Mounting](NFS-Client-Setup-and-Mounting.md) — end-to-end client mount workflow these options plug into
- [NFS-Exports-Configuration](NFS-Exports-Configuration.md) — the server-side options each mount negotiates against
- [NFS-with-Kerberos](NFS-with-Kerberos.md) — using `sec=krb5`/`krb5i`/`krb5p` mount flavors
- [NFS-Security-Hardening](NFS-Security-Hardening.md) — `nosuid`/`nodev`, version pinning, and firewalling mounts
- [Linux Administration & Server Hardening](../Readme.md) — Linux administration & server hardening hub
