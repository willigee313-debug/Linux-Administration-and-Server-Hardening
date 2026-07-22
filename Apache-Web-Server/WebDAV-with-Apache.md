# WebDAV with Apache

## Overview

**WebDAV** (Web Distributed Authoring and Versioning) extends the HTTP protocol with methods for remote file management — `PUT`, `DELETE`, `MKCOL`, `MOVE`, `COPY`, `PROPFIND`, and `LOCK`. It turns a web server into a read/write file store reachable over HTTP or HTTPS. Apache implements WebDAV through the `mod_dav` and `mod_dav_fs` modules.

This note walks through enabling WebDAV on a RHEL-family Apache (`httpd`) server, exposing a directory for authoring, testing it with `curl`/`cadaver`/`nmap`, and locking it down with HTTP Basic authentication over TLS.

> [!WARNING]
> A world-writable WebDAV endpoint that accepts `PUT` is a classic remote-code-execution vector: an attacker who can upload a `.php` file into the document root can often execute it. The exact `PUT`/`DELETE` flow shown here for learning is the same one covered offensively in Web-Enumeration. Never expose an unauthenticated, plaintext WebDAV share on an untrusted network.

## Features of WebDAV

- Access and manage files over HTTP/S.
- Supports creation, editing, moving, and deletion of files.
- Integrates with version control tools like Git and SVN.
- Clients can mount WebDAV directories as local drives.
- Suitable for collaborative environments and remote file hosting.

## Architecture

```mermaid
flowchart LR
    A[WebDAV client<br/>curl / cadaver / OS mount] -->|PUT / GET / DELETE / PROPFIND| B[Apache httpd :80/:443]
    B --> C[mod_dav<br/>protocol engine]
    C --> D[mod_dav_fs<br/>filesystem backend]
    D --> E[/var/www/html/webdav]
    B -. Basic Auth .-> F[/etc/httpd/htpasswd]
```

`mod_dav` speaks the WebDAV protocol; `mod_dav_fs` maps those operations onto the local filesystem under the `DAV On` directory. Optional Basic authentication is checked against an `htpasswd` file before any method is allowed.

## Install and Verify Apache

Check if Apache is installed — verifies whether Apache is already present:

```bash
rpm -qa | grep httpd
```

Install Apache and related packages on RHEL-based systems:

```bash
yum install httpd*
```

List all currently active modules:

```bash
httpd -M
```

Check for WebDAV module support:

```bash
httpd -M | grep fs
```

Expected output:

```text
dav_fs_module (shared)
```

This confirms that WebDAV support is enabled via `mod_dav_fs`.

## Configuration

### Create and Set Up the WebDAV Directory

Create the directory that will hold WebDAV content:

```bash
mkdir /var/www/html/webdav
```

Set ownership so Apache owns the directory:

```bash
chown apache:apache /var/www/html/webdav/
```

Set permissions:

```bash
chmod 777 /var/www/html/webdav/
```

> [!WARNING]
> `777` (world read/write/execute) is used here only to get the lab working quickly. In production prefer `755` or `775`, and rely on the `apache:apache` ownership plus authentication for write access — a `777` directory in the document root is an open invitation for arbitrary file upload.

### Configure Apache for WebDAV

Edit the Apache configuration:

```bash
vim /etc/httpd/conf/httpd.conf
```

Add or update the Virtual Host block:

```apache
<VirtualHost *:80>
    DocumentRoot /var/www/html/
    DirectoryIndex index.html
    <Directory /var/www/html/webdav>
        DAV On
    </Directory>
</VirtualHost>
```

This enables WebDAV under the `/webdav` path.

### Test WebDAV Directory Access

Use `curl` to send an `OPTIONS` request — this reveals the HTTP methods the server allows:

```bash
curl -v -X OPTIONS http://192.168.1.32/webdav/
```

A WebDAV-enabled endpoint advertises the extended methods (`PUT`, `DELETE`, `PROPFIND`, `MKCOL`, `MOVE`, `COPY`, `LOCK`, `UNLOCK`) in the `Allow` and `DAV` response headers.

> [!NOTE]
> **📸 Screenshot**
> _Capture: curl -v OPTIONS response showing the DAV header and the Allow line listing PUT, DELETE, PROPFIND and other WebDAV methods_

### Configure SELinux and Firewall

Restart Apache to load the new configuration:

```bash
systemctl restart httpd.service
```

Check SELinux status:

```bash
sestatus
```

Disable SELinux temporarily (not recommended for production):

```bash
vim /etc/sysconfig/selinux
```

Change the value:

```bash
SELINUX=disabled
```

> [!TIP]
> Instead of disabling SELinux, keep it enforcing and grant Apache write access to the WebDAV directory with the correct context label. This preserves host protection while still allowing uploads:

```bash
chcon -R -t httpd_sys_content_t /var/www/html/webdav
```

> [!NOTE]
> For WebDAV to *write* under enforcing SELinux you typically need the writable label `httpd_sys_rw_content_t` (via `semanage fcontext -a -t httpd_sys_rw_content_t '/var/www/html/webdav(/.*)?' && restorecon -Rv /var/www/html/webdav`). The `httpd_sys_content_t` label shown above grants read/serve access.

## Examples

### Uploading and Deleting Files Using curl

Upload a text file:

```bash
curl -X PUT -d "hi" http://192.168.1.32/webdav/1.txt
```

Upload a PHP file:

```bash
curl -X PUT -d "<?php phpinfo(); ?>" http://192.168.1.32/webdav/phpinfo.php
```

Delete files:

```bash
curl -X DELETE http://192.168.1.32/webdav/phpinfo.php
```

```bash
curl -v -X DELETE http://192.168.1.32/webdav/1.txt
```

> [!WARNING]
> The `phpinfo.php` upload above demonstrates exactly why an unauthenticated WebDAV `PUT` is dangerous: if the document root executes PHP, that uploaded file runs server-side code. Always require authentication (below) before enabling `DAV On` on an internet-facing host.

## Secure WebDAV with Basic Authentication

### Create the User Password File

Create the file and add the first user (`-c` creates the file — use it only once):

```bash
htpasswd -c /etc/httpd/htpasswd dev
```

Add more users (no `-c`, so the existing file is preserved):

```bash
htpasswd /etc/httpd/htpasswd admin
```

```bash
htpasswd /etc/httpd/htpasswd root
```

```bash
htpasswd /etc/httpd/htpasswd user
```

View the `.htpasswd` file:

```bash
cat /etc/httpd/htpasswd
```

Sample output:

```text
dev:$apr1$GXmFNd4f$HBa.Fu2SB1SMUncBcATuG/
admin:$apr1$NmNcn0p5$Lhk2I/HVjXV7mKKpXis4C/
root:$apr1$Qey..Fqn$y2GIMsuP1JDDMrnWHgDnU.
user:$apr1$zp/J6/mB$v.h.deWHBq/34CABimwk3/
```

> [!NOTE]
> `$apr1$` is Apache's MD5-based hash. It is acceptable for `htpasswd` files but weak by modern standards — prefer `htpasswd -B` (bcrypt) for new deployments.

### Configure Apache for WebDAV Authentication

Edit the Apache config again:

```bash
vim /etc/httpd/conf/httpd.conf
```

Update the VirtualHost block to require a valid user:

```apache
<VirtualHost *:80>
    DocumentRoot /var/www/html/
    DirectoryIndex index.html
    <Directory /var/www/html/webdav>
        DAV On
        AuthType Basic
        AuthName "webdav"
        AuthUserFile /etc/httpd/htpasswd
        Require valid-user
    </Directory>
</VirtualHost>
```

Restart Apache after the config change:

```bash
systemctl restart httpd.service
```

### Notes for Production Environments

- **Always use HTTPS** to secure credentials in transit.
- Use SELinux context labeling instead of disabling it.
- Apply firewall rules or IP whitelisting for added security.

```bash
chcon -R -t httpd_sys_content_t /var/www/html/webdav
```

> [!IMPORTANT]
> Basic authentication sends the username and password Base64-encoded — **not encrypted**. Over plain HTTP those credentials are trivially sniffed. Pair `Require valid-user` with TLS (`<VirtualHost *:443>` + `mod_ssl`) so the whole exchange is encrypted. See [Multiple-Web-Sites-with-SSL](Multiple-Web-Sites-with-SSL.md) and [Binding-with-Type(SSL-TLS)](Binding-with-Type(SSL-TLS).md).

## WebDAV Client Testing

Test the HTTP methods over HTTP and HTTPS:

```bash
curl -v -X OPTIONS http://192.168.1.32/webdav/
```

```bash
curl -i -k -X OPTIONS https://192.168.1.32/webdav/
```

Scan WebDAV with Nmap to enumerate allowed methods:

```bash
nmap -v -sT -sV -A -O -p 80 --script=http-methods.nse --script-args http-methods.url-path='/webdav/' 192.168.1.32
```

## Authenticated File Operations

Send `OPTIONS` with credentials:

```bash
curl -v -u dev:123 -X OPTIONS http://192.168.1.32/webdav/
```

```bash
curl -i -k -u dev:123 -X OPTIONS https://192.168.1.32/webdav/
```

Authenticated file upload:

```bash
curl -u dev:123 -X PUT -d "hi" http://192.168.1.32/webdav/1.txt
```

```bash
curl -X PUT -u dev:123 -d "<?php phpinfo(); ?>" http://192.168.1.32/webdav/phpinfo.php
```

Authenticated file deletion:

```bash
curl -u dev:123 -X DELETE http://192.168.1.32/webdav/phpinfo.php
```

```bash
curl -v -u root:123 -X DELETE http://192.168.1.32/webdav/1.txt
```

## Using cadaver — WebDAV Command-Line Client

`cadaver` is an interactive shell-style WebDAV client, convenient for browsing and bulk operations.

Install cadaver:

```bash
apt install cadaver
```

or

```bash
yum install cadaver
```

Connect to the WebDAV server:

```bash
cadaver http://192.168.1.32/webdav/
```

Sample `cadaver` session:

```text
dav:/webdav/> ?
```

```text
dav:/webdav/> ls
```

```text
dav:/webdav/> put network.png
```

```text
dav:/webdav/> mkdir newdir
```

```text
dav:/webdav/> mput *.php
```

```text
dav:/webdav/> delete *
```

Connect to a different path:

```bash
cadaver http://192.168.1.32/test/
```

## Security Considerations

- **Authenticate every write.** `DAV On` with no `Require` directive lets anyone `PUT`/`DELETE`. Always pair it with `Require valid-user`.
- **Encrypt the channel.** Basic auth over HTTP leaks credentials; terminate WebDAV on TLS (`:443`) so both the login and the file contents are protected.
- **Block executable uploads.** Keep the WebDAV directory outside any path where PHP/CGI executes, or disable script handling for it, so an uploaded `phpinfo.php` cannot run.
- **Least-privilege filesystem.** Use `755`/`775` and `apache:apache` ownership rather than `777`; grant the writable SELinux label only to the WebDAV directory.
- **Restrict exposure.** Firewall or IP-whitelist the endpoint so only trusted authoring hosts can reach it.

## Troubleshooting

| Symptom | Likely cause | Check / fix |
|---|---|---|
| `403 Forbidden` on `PUT` | SELinux blocking writes or wrong permissions | Apply `httpd_sys_rw_content_t`; verify `apache:apache` ownership |
| `OPTIONS` shows no WebDAV methods | `mod_dav_fs` not loaded or `DAV On` missing | `httpd -M \| grep dav`; confirm the `<Directory>` block |
| `401 Unauthorized` with correct password | `AuthUserFile` path wrong or user not in file | `cat /etc/httpd/htpasswd`; re-check the `AuthUserFile` line |
| Credentials work over HTTP but leak | Basic auth without TLS | Move the VirtualHost to `:443` with `mod_ssl` |
| Config change not taking effect | Apache not reloaded | `httpd -t` then `systemctl restart httpd.service` |

## References

- [Apache `mod_dav`](https://httpd.apache.org/docs/current/mod/mod_dav.html) — WebDAV protocol module
- [Apache `mod_dav_fs`](https://httpd.apache.org/docs/current/mod/mod_dav_fs.html) — filesystem provider
- [RFC 4918 — HTTP Extensions for WebDAV](https://www.rfc-editor.org/rfc/rfc4918) — the WebDAV specification
- [cadaver](http://www.webdav.org/cadaver/) — command-line WebDAV client
- OWASP Testing Guide — testing for HTTP methods and WebDAV

## Related

- [Apache-Web-Server-Setup-and-Configuration](Apache-Web-Server-Setup-and-Configuration.md) — base httpd config WebDAV plugs into
- [Binding-with-Type(SSL-TLS)](Binding-with-Type(SSL-TLS).md) — serve WebDAV over TLS so Basic auth is encrypted
- [Multiple-Web-Sites-with-SSL](Multiple-Web-Sites-with-SSL.md) — TLS virtual host patterns for authoring endpoints
- [CGI-Scripts(Common-Gateway-Interface)](CGI-Scripts(Common-Gateway-Interface).md) — another Apache server-side feature
- Web-Enumeration — WebDAV PUT/methods are an exploitation vector
- [Linux Administration & Server Hardening](../Readme.md) — Linux administration & server hardening hub
