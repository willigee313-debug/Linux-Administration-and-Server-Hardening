# Apache Web Server

Installing and configuring Apache HTTPD: virtual hosts, SSL/TLS, CGI, and PHP.

> [!NOTE]
> **Module hub**
> Part of the **[Linux Administration & Server Hardening](../Readme.md)** course.

## Overview

Hosting websites with the Apache HTTP Server. This module covers installation on Debian, the several kinds of binding (IP, port, domain name, and SSL/TLS), multiple SSL-enabled virtual hosts, directory listing and access control, user home directories, CGI, WebDAV, and a PHP/MySQL stack — a complete practical foundation for secure web hosting.

## Learning Objectives

By the end of this module you will be able to:

- Install Apache and serve multiple sites with name-, IP-, and port-based virtual hosts
- Enable HTTPS with SSL/TLS for one or many sites
- Configure access control, CGI, WebDAV, and a PHP/MySQL application stack

## Topics Covered

This module contains **13 notes**.

| Note | Topic |
| --- | --- |
| [Apache-Directory-Listing-and-Access-Control](Apache-Directory-Listing-and-Access-Control.md) | Apache Directory Listing and Access Control |
| [Apache-Web-Server-Setup-and-Configuration](Apache-Web-Server-Setup-and-Configuration.md) | Apache Web Server Setup and Configuration |
| [Apache2-Setup-on-Debian](Apache2-Setup-on-Debian.md) | Apache2 Setup on Debian |
| [Binding-with-Domain-Name](Binding-with-Domain-Name.md) | Binding with Domain Name |
| [Binding-with-IP-Add-in-Apache](Binding-with-IP-Add-in-Apache.md) | Binding with IP Add in Apache |
| [Binding-with-Port-No.-in-Apache](Binding-with-Port-No.-in-Apache.md) | Binding with Port No. in Apache |
| [Binding-with-Type(SSL-TLS)](Binding-with-Type(SSL-TLS).md) | Binding with Type(SSL TLS) |
| [CGI-Scripts(Common-Gateway-Interface)](CGI-Scripts(Common-Gateway-Interface).md) | CGI Scripts(Common Gateway Interface) |
| [Enable-Apache-User-Home-Directories](Enable-Apache-User-Home-Directories.md) | Enable Apache User Home Directories |
| [Multiple-Web-Sites-with-SSL](Multiple-Web-Sites-with-SSL.md) | Multiple Web Sites with SSL |
| [PHP-and-Mysql-Installation-and-Configuration](PHP-and-Mysql-Installation-and-Configuration.md) | PHP and Mysql Installation and Configuration |
| [Types-of-Binding](Types-of-Binding.md) | Types of Binding |
| [WebDAV-with-Apache](WebDAV-with-Apache.md) | WebDAV with Apache |

## Practical Labs

> [!NOTE]
> **Hands-on labs**
> Guided, reproducible labs for this topic live in the course-wide **[Practical Labs](../Practical-Labs/Readme.md)** collection — build them on a disposable VM. The step-by-step configuration walkthroughs in the notes above also work as self-paced labs.

## Best Practices

- Disable unneeded modules and the default site to shrink attack surface
- Redirect HTTP to HTTPS and enable HSTS on production sites
- Keep each site's config in its own vhost file for clarity

## Security Considerations

> [!WARNING]
> **Harden before you expose**
> Apply these controls before placing any service on an untrusted network.

- Terminate TLS with modern ciphers and automate certificate renewal
- Disable directory listing (`Options -Indexes`) unless explicitly required
- Run CGI/PHP with least privilege and keep the server patched

## Troubleshooting

| Symptom | Likely cause & fix |
| --- | --- |
| Apache will not start after a config change | Run `apachectl configtest` to find the offending directive |
| TLS site shows the wrong certificate | Check `ServerName`/SNI and that the correct vhost owns the certificate |

## References

- [Apache HTTP Server documentation](https://httpd.apache.org/docs/)
- [Mozilla SSL Configuration Generator](https://ssl-config.mozilla.org/)
- [OWASP Secure Headers Project](https://owasp.org/www-project-secure-headers/)

## Related Notes

- [Linux Administration & Server Hardening](../Readme.md) — course hub and map of content
- [Containers](../Containers/Readme.md) — running the web stack in Docker/Podman
- [Secure Web Hosting (Project 02)](../Enterprise-Projects/Project-02-Secure-Web-Hosting-Platform.md) — capstone build
- [Security, Firewall and Monitoring](../Security-Firewall-and-Monitoring/Readme.md) — related module
- [Domain Name System (DNS)](../Domain-Name-System-DNS/Readme.md) — related module
- [Proxy Server (Squid)](../Proxy-Server-Squid/Readme.md) — related module
