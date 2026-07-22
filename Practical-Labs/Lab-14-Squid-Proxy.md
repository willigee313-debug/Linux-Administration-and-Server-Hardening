# Lab 14 — Squid Proxy

## Objective

Deploy Squid as a forward (caching) proxy that mediates outbound HTTP/HTTPS traffic for a small internal network. The learner will build source/destination ACLs to enforce an allow/deny policy, add HTTP Basic authentication with `htpasswd` so only credentialed users can browse, confirm object caching is actually happening (cache HIT vs MISS), and validate the whole chain from a client using both a browser and `curl`. This lab operationalizes the concepts in [Readme](../Proxy-Server-Squid/Readme.md) and complements firewall egress-control labs in [Readme](../Readme.md).

## Requirements

| Host | Role | OS | IP | Resources |
|---|---|---|---|---|
| `proxy01` | Squid forward proxy | RHEL 9 / Rocky 9 (or Debian 12) | `192.168.56.10` | 1 vCPU, 1 GB RAM, 10 GB disk |
| `client01` | Test client (browser + curl) | Debian 12 / Ubuntu 22.04 | `192.168.56.20` | 1 vCPU, 1 GB RAM |
| — | Internet egress via NAT (Vbox/host adapter) on `proxy01` only | — | — | — |

This lab assumes **RHEL-family (`dnf`, firewalld, SELinux)** as the primary path, with **Debian-family (`apt`, ufw)** commands called out wherever they diverge. Client can be either family — only a browser/curl matters.

> [!IMPORTANT]
> **Network isolation**
> `client01` should have **no default route to the internet** except through `proxy01` (or simulate this by only trusting the proxy for policy testing). This proves traffic is genuinely proxied, not routed around it.

## Topology

```mermaid
flowchart LR
    subgraph LAN["Internal LAN 192.168.56.0/24"]
        C[client01<br/>192.168.56.20<br/>browser + curl]
    end
    subgraph DMZ["Proxy Host"]
        P[proxy01<br/>Squid 192.168.56.10<br/>ports 3128 / auth]
    end
    I((Internet))

    C -- "HTTP_PROXY=192.168.56.10:3128" --> P
    P -- "allowed domains only<br/>after ACL + auth check" --> I
    P -. "cache_dir /var/spool/squid<br/>100 MB" .-> P
```

## Setup

### 1. Install Squid

```bash
# RHEL / Rocky / AlmaLinux
sudo dnf install -y squid httpd-tools

# Debian / Ubuntu
sudo apt update && sudo apt install -y squid apache2-utils
```

```bash
sudo systemctl enable --now squid
systemctl status squid --no-pager
```

### 2. Open the firewall

```bash
# RHEL family (firewalld)
sudo firewall-cmd --permanent --add-port=3128/tcp
sudo firewall-cmd --reload

# Debian family (ufw)
sudo ufw allow 3128/tcp
```

> [!WARNING]
> **Don't expose 3128 to the internet**
> An open proxy with no ACLs/auth becomes a relay for abuse within minutes of being scanned. Always bind ACLs (and ideally a firewall source restriction) before enabling the service on anything internet-facing.

### 3. Back up the stock config

```bash
sudo cp /etc/squid/squid.conf /etc/squid/squid.conf.orig
```

### 4. Define ACLs — source network, safe ports, and a deny-list

Edit `/etc/squid/squid.conf`. Squid evaluates `http_access` rules **top to bottom, first match wins** — order matters.

```conf
# --- ACL definitions ---
acl localnet src 192.168.56.0/24
acl SSL_ports port 443
acl Safe_ports port 80 443 21 8080
acl CONNECT method CONNECT

# Deny-list of blocked destination domains (one per line, no protocol/path)
acl blocked_sites dstdomain "/etc/squid/blocked_sites.txt"

# Authenticated users group
acl auth_users proxy_auth REQUIRED

# --- Auth scheme ---
auth_param basic program /usr/lib64/squid/basic_ncsa_auth /etc/squid/passwords
auth_param basic realm Proxy Authentication Required
auth_param basic credentialsttl 2 hours

# --- Access rules (order matters) ---
http_access deny !Safe_ports
http_access deny CONNECT !SSL_ports
http_access deny blocked_sites
http_access allow localnet auth_users
http_access deny all
```

> [!NOTE]
> **Path to `basic_ncsa_auth` varies**
> RHEL/Rocky: `/usr/lib64/squid/basic_ncsa_auth`. Debian/Ubuntu: `/usr/lib/squid/basic_ncsa_auth`. Run `sudo find / -name basic_ncsa_auth 2>/dev/null` if unsure and fix the path before restarting.

Create the deny-list and the password file:

```bash
sudo tee /etc/squid/blocked_sites.txt <<'EOF'
.facebook.com
.tiktok.com
EOF

sudo htpasswd -c -B /etc/squid/passwords labuser
# -c creates the file (omit -c when adding a 2nd user), -B forces bcrypt
sudo chown squid:squid /etc/squid/passwords
sudo chmod 640 /etc/squid/passwords
```

### 5. Enable caching

Add (or confirm) the cache directives:

```conf
# --- Caching ---
cache_dir ufs /var/spool/squid 100 16 256
maximum_object_size 10 MB
cache_mem 64 MB
```

Initialize the cache swap directories (required the first time `cache_dir` is added):

```bash
sudo systemctl stop squid
sudo squid -z
sudo systemctl start squid
```

### 6. Validate config syntax and restart

```bash
sudo squid -k parse
sudo systemctl restart squid
sudo systemctl status squid --no-pager
```

> [!WARNING]
> **SELinux on RHEL**
> If Squid fails to read `/etc/squid/passwords` or the deny-list with `Permission denied` in `/var/log/squid/cache.log`, check `getenforce`. Restore correct context rather than disabling SELinux:
> ```bash
> sudo restorecon -Rv /etc/squid
> ```

### 7. Point the client at the proxy

On `client01`, for CLI tools:

```bash
export http_proxy="http://labuser:PASSWORD@192.168.56.10:3128/"
export https_proxy="http://labuser:PASSWORD@192.168.56.10:3128/"
```

For a browser (Firefox example): Settings → Network Settings → Manual proxy configuration → HTTP Proxy `192.168.56.10`, Port `3128`, check "Also use this proxy for HTTPS".

> [!NOTE]
> **📸 Screenshot**
> _Capture: Firefox manual proxy configuration dialog showing 192.168.56.10:3128 entered, alongside a terminal showing `curl -x` returning HTTP 200 through the proxy._

## Commands — quick reference

```bash
# Tail live access log while testing
sudo tail -f /var/log/squid/access.log

# Reload config without dropping the cache
sudo squid -k reconfigure

# Force full restart (rebuilds swap state)
sudo systemctl restart squid
```

## Validation

**1. Unauthenticated request is rejected (407):**

```bash
curl -x http://192.168.56.10:3128 -I http://example.com
```
```text
HTTP/1.1 407 Proxy Authentication Required
```

**2. Authenticated request to an allowed site succeeds (200):**

```bash
curl -x http://labuser:PASSWORD@192.168.56.10:3128 -I http://example.com
```
```text
HTTP/1.1 200 OK
X-Cache: MISS from proxy01
```

**3. Blocked domain is denied (403):**

```bash
curl -x http://labuser:PASSWORD@192.168.56.10:3128 -I http://www.facebook.com
```
```text
HTTP/1.1 403 Forbidden
```

**4. Caching works — second request is a HIT:**

```bash
curl -x http://labuser:PASSWORD@192.168.56.10:3128 -o /dev/null -s -w "%{http_code}\n" http://example.com
curl -x http://labuser:PASSWORD@192.168.56.10:3128 -I http://example.com
```
```text
HTTP/1.1 200 OK
X-Cache: HIT from proxy01
```

**5. Confirm in the access log which rule matched:**

```bash
sudo tail -n 5 /var/log/squid/access.log
```
```text
1690000012.345    120 192.168.56.20 TCP_MISS/200 512 GET http://example.com/ labuser HIER_DIRECT/93.184.216.34 text/html
1690000015.210      1 192.168.56.20 TCP_MEM_HIT/200 512 GET http://example.com/ labuser HIER_NONE/- text/html
1690000020.001      0 192.168.56.20 TCP_DENIED/403 3919 GET http://www.facebook.com/ labuser NONE/- text/html
```

**6. Verify cache disk usage exists:**

```bash
sudo du -sh /var/spool/squid
```
```text
2.1M    /var/spool/squid
```

## Cleanup

```bash
# Remove firewall rule
sudo firewall-cmd --permanent --remove-port=3128/tcp && sudo firewall-cmd --reload   # RHEL
sudo ufw delete allow 3128/tcp                                                        # Debian

# Stop and disable Squid
sudo systemctl disable --now squid

# Wipe cache and lab-specific config artifacts
sudo rm -rf /var/spool/squid/*
sudo rm -f /etc/squid/blocked_sites.txt /etc/squid/passwords

# Restore original config
sudo cp /etc/squid/squid.conf.orig /etc/squid/squid.conf

# Unset client proxy env vars
unset http_proxy https_proxy
```

## Troubleshooting

| Symptom | Likely Cause | Fix |
|---|---|---|
| `407` even with correct credentials | Wrong path to `basic_ncsa_auth` in `auth_param` | `find / -name basic_ncsa_auth`, correct the path, `squid -k reconfigure` |
| `squid -z` hangs or errors on swap dirs | `cache_dir` path doesn't exist / wrong permissions | `sudo mkdir -p /var/spool/squid && sudo chown squid:squid /var/spool/squid` then re-run `-z` |
| Client gets connection refused | Firewall not opened, or Squid bound to `127.0.0.1` only | Check `http_port 3128` (no explicit bind to localhost) and firewall rule |
| Every request is `TCP_MISS`, never `HIT` | Site sends `Cache-Control: no-store` / `no-cache`, or object too small/large for policy | Test against a static, cache-friendly site like `http://example.com`; check `refresh_pattern` |
| `Permission denied` reading passwords file (RHEL only) | SELinux context mismatch after manual file creation | `sudo restorecon -Rv /etc/squid` |
| Config changes don't take effect | Used `reconfigure` after a directive that requires full restart (e.g. new `cache_dir`) | `sudo systemctl restart squid` instead of `-k reconfigure` |

## References

- Squid Official Documentation — [http_access](http://www.squid-cache.org/Doc/config/http_access/) and [cache_dir](http://www.squid-cache.org/Doc/config/cache_dir/) directives
- `man squid.conf`
- RHEL 9 Networking Guide — Configuring a caching-only web proxy server (Red Hat customer portal)
- CIS Squid / Proxy Server hardening benchmarks (network egress control)

## Related Notes

- [Readme](../Proxy-Server-Squid/Readme.md) — Squid proxy module home
- [Lab 13 — Firewalld / Nftables](Lab-07-Firewall-Configuration.md) — pairs with this lab for full egress-control policy
- [Linux Administration & Server Hardening](../Readme.md) — course hub and map of content
