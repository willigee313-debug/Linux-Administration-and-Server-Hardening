# Flashcards — Infrastructure Services

Spaced-repetition cards covering DNS, DHCP, Apache, Samba, NFS, LDAP, Squid, and FTP/vsftpd administration — drawn from the Linux Administration & Server Hardening infrastructure-services modules. Cards favor commands, config directives, and file paths likely to appear on RHCSA/LFCS/Linux+/LPIC-1 style exams.

## Domain Name System (DNS)

Which BIND directive disables recursive resolution on an authoritative nameserver, preventing it from being abused as an open resolver?::`recursion no;`
Which command validates the syntax of `/etc/named.conf` before restarting BIND?::`named-checkconf`
Which dig query type performs a full zone transfer, useful for enumerating every record in a zone?::AXFR — `dig AXFR <domain> @<server>`
Which `named.conf` directive on a master zone restricts which secondary servers may pull zone transfers?::`allow-transfer { <slave-ip>; };`

## DHCP

What are the four packets in the DHCP DORA exchange, in order?::DHCPDISCOVER, DHCPOFFER, DHCPREQUEST, DHCPACK
On which UDP port does the DHCP server listen for client requests (as opposed to the client's port)?::67/udp (clients use 68/udp)
Which `dhcpd.conf` block reserves a fixed IP address for a specific MAC address?::`host <name> { hardware ethernet <mac>; fixed-address <ip>; }`

## Apache Web Server

Which `Options` directive disables Apache's automatic directory listing (autoindex)?::`Options -Indexes`
Which `htpasswd` flag must be used only for the very first user when creating a new password file?::`-c` (it creates/overwrites the file — omit it for subsequent users)
Which command tests Apache's configuration syntax before restarting the service?::`httpd -t`
Which `Options` directive must be set on a directory to allow CGI script execution?::`Options +ExecCGI`

## Samba (SMB/CIFS)

Which command creates a Samba password for an existing Linux user account?::`smbpasswd -a <user>`
Which two `smb.conf` directives together force every connection on a share to be treated as an unauthenticated guest?::`guest ok = yes` and `guest only = yes`
Which command validates `smb.conf` syntax before restarting `smb.service`?::`testparm`

## NFS Server

In `/etc/exports`, why does `192.168.1.0/24 (rw)` (with a space before the parenthesis) misbehave?::The `(rw)` options apply to an implicit `*` (everyone), not just that subnet — the whitespace before `(` matters
Which command re-exports all entries in `/etc/exports` without restarting the NFS service?::`exportfs -ra`
Which NFS export option strips the client's root user of root privileges on the exported filesystem?::`root_squash`

## LDAP Server

What port does LDAPS (TLS-from-connection-start) use, as opposed to plain/StartTLS LDAP?::636 (plain/StartTLS uses 389)
Which command dumps an OpenLDAP database's contents directly from the backend files?::`slapcat`
Which `ldapsearch` flag requires StartTLS to succeed rather than silently falling back to plaintext?::`-ZZ`

## Squid Proxy

What TCP port does Squid listen on by default?::3128
In Squid's `http_access` rule list, which rule actually governs a given request?::The first rule that matches, evaluated top to bottom — first match wins
Which command validates `squid.conf` syntax without restarting the service?::`squid -k parse`
Which `http_port` directive enables transparent (intercepting) proxy mode?::`http_port 3128 transparent`

## FTP (vsftpd)

Which two vsftpd files must both have `root` removed/commented out to permit root FTP login?::`/etc/vsftpd/ftpusers` and `/etc/vsftpd/user_list`
Which `vsftpd.conf` directive permits anonymous users to upload files?::`anon_upload_enable=YES`
Which two `vsftpd.conf` directives enable TLS/FTPS by pointing to the certificate and private key (alongside `ssl_enable=YES`)?::`rsa_cert_file` and `rsa_private_key_file`
Which `vsftpd.conf` directive sets a shared directory that all local users are placed and chrooted into?::`local_root=/backup` (combined with `chroot_local_user=YES`)

## Related
- [Domain Name System (DNS)](../Domain-Name-System-DNS/Readme.md)
- [Dynamic Host Configuration Protocol (DHCP)](../Dynamic-Host-Configuration-Protocol-DHCP/Readme.md)
- [Apache Web Server](../Apache-Web-Server/Readme.md)
- [Samba (SMB/CIFS) Server](../Samba-SMB-CIFS-Server/Readme.md)
- [NFS Server](../NFS-Server/Readme.md)
- [LDAP Server](../LDAP-Server/Readme.md)
- [Proxy Server (Squid)](../Proxy-Server-Squid/Readme.md)
- [FTP Server (vsftpd)](../FTP-Server-VSFTPD/Readme.md)
- [Exam Preparation](Readme.md)
- [Linux Administration & Server Hardening](../Readme.md)
