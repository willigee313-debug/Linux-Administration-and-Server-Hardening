# OpenVPN on OpenWrt

## Overview

This guide walks through installing, configuring, and managing **OpenVPN** on OpenWrt. It supports both **client** and **server** setups and uses UCI sections that reference either a `.ovpn` profile (client) or a hand-written `.conf` file (server). Firewall integration, useful diagnostic commands, and the optional LuCI workflow are included.

## Prerequisites

- OpenWrt device with internet access
- Sufficient storage (OpenVPN can be large for embedded routers)
- SSH or LuCI web access
- CA certs and keys (for manual/server setup) or a `.ovpn` file (for client setup)

## Architecture

```mermaid
flowchart LR
    C[Remote client] -->|UDP 1194| W((wan))
    W --> O[openvpn instance<br/>tun0]
    O --> V[vpn zone]
    V <-->|forwarding| L[lan zone]
    L --> H[LAN hosts<br/>192.168.1.0/24]
```

## Configuration

### 1. Install OpenVPN Packages

Update the package list and install OpenVPN with its LuCI front-end:

```bash
opkg update
opkg install openvpn-openssl luci-app-openvpn
```

Optional (for certificate generation / `.ovpn` handling):

```bash
opkg install openvpn-easy-rsa
```

### 2. OpenVPN as Client (using `.ovpn`)

#### Step 1: Upload the `.ovpn` file

Transfer the `.ovpn` file to your router:

```bash
scp myvpn.ovpn root@192.168.1.1:/etc/openvpn/
```

#### Step 2: Reference it from UCI

Create a config in UCI pointing at the `.ovpn` profile:

```bash
uci set openvpn.myvpn="openvpn"
uci set openvpn.myvpn.enabled='1'
uci set openvpn.myvpn.config='/etc/openvpn/myvpn.ovpn'
uci commit openvpn
```

#### Step 3: Start and Enable the Service

```bash
/etc/init.d/openvpn restart
/etc/init.d/openvpn enable
```

#### Step 4: Verify the Connection

```bash
logread -e openvpn
ip a
```

### 3. OpenVPN as Server (manual setup)

#### Step 1: Generate Certificates

If needed, use `easy-rsa`:

```bash
mkdir -p /etc/easy-rsa
cp -r /usr/share/easy-rsa/* /etc/easy-rsa/
cd /etc/easy-rsa
./easyrsa init-pki
./easyrsa build-ca
./easyrsa build-server-full server nopass
./easyrsa build-client-full client1 nopass
./easyrsa gen-dh
```

Copy the resulting files to `/etc/openvpn/`.

#### Step 2: Server Configuration

Create the file `/etc/openvpn/server.conf`:

```conf
port 1194
proto udp
dev tun
ca /etc/openvpn/ca.crt
cert /etc/openvpn/server.crt
key /etc/openvpn/server.key
dh /etc/openvpn/dh.pem
server 10.8.0.0 255.255.255.0
persist-key
persist-tun
keepalive 10 120
cipher AES-256-CBC
verb 3
push "redirect-gateway def1"
push "dhcp-option DNS 8.8.8.8"
```

#### Step 3: UCI Configuration

```bash
uci set openvpn.myserver="openvpn"
uci set openvpn.myserver.enabled='1'
uci set openvpn.myserver.config='/etc/openvpn/server.conf'
uci commit openvpn
/etc/init.d/openvpn restart
```

### 4. Firewall Rules

#### Add an OpenVPN Zone (optional)

```bash
uci add firewall zone
uci set firewall.@zone[-1].name='vpn'
uci set firewall.@zone[-1].input='ACCEPT'
uci set firewall.@zone[-1].output='ACCEPT'
uci set firewall.@zone[-1].forward='REJECT'
uci set firewall.@zone[-1].device='tun0'
uci commit firewall
```

#### Allow Forwarding Between VPN and LAN

```bash
uci add firewall forwarding
uci set firewall.@forwarding[-1].src='vpn'
uci set firewall.@forwarding[-1].dest='lan'
uci add firewall forwarding
uci set firewall.@forwarding[-1].src='lan'
uci set firewall.@forwarding[-1].dest='vpn'
uci commit firewall
/etc/init.d/firewall restart
```

## Commands

| Task | Command |
| --- | --- |
| Check the VPN interface | `ip a show tun0` |
| View OpenVPN logs | `logread -e openvpn` |
| Restart the VPN service | `/etc/init.d/openvpn restart` |

### Check the VPN Interface

```bash
ip a show tun0
```

### View Logs

```bash
logread -e openvpn
```

### Restart the VPN Service

```bash
/etc/init.d/openvpn restart
```

## LuCI Web Interface (optional)

- Go to **Services > OpenVPN**
- Add an instance (manual or import `.ovpn`)
- Enable and start it
- Watch logs from **Status > System Log**

> [!NOTE]
> **📸 Screenshot**
> _Capture: OpenWrt LuCI Services OpenVPN page listing the myvpn instance with an enabled checkbox and a Start button_

## Security Considerations

> [!WARNING]
> **`AES-256-CBC` is legacy**
> The `cipher AES-256-CBC` directive above matches the source configuration, but modern OpenVPN (2.5+) prefers AEAD ciphers negotiated via `data-ciphers` (e.g. `AES-256-GCM`). Prefer GCM where both ends support it, and never reuse keys across deployments.

- Protect private keys: `/etc/openvpn/*.key` and `*.conf` should be readable only by root.
- Keep the `vpn` zone `forward` policy at `REJECT` and open only the specific `lan`⇄`vpn` forwardings you need.
- Restrict the inbound WAN rule to UDP 1194 only; do not broaden the source unless required.
- Rotate CA and client certificates periodically and revoke lost client credentials.

## Troubleshooting

- Ensure `/etc/openvpn/*.conf` has proper permissions.
- Check if the tun module is loaded: `lsmod | grep tun`
- Firewall not allowing UDP 1194? Add a port rule:

```bash
uci add firewall rule
uci set firewall.@rule[-1].name='Allow-OpenVPN-Inbound'
uci set firewall.@rule[-1].src='wan'
uci set firewall.@rule[-1].target='ACCEPT'
uci set firewall.@rule[-1].proto='udp'
uci set firewall.@rule[-1].dest_port='1194'
uci commit firewall
/etc/init.d/firewall restart
```

## Related
- [Linux Administration & Server Hardening](../Readme.md) — Linux administration & server hardening hub.
- [Router and Firewall OS (OpenWrt)](Readme.md) — module hub.
- [Firewall-Rules-in-OpenWrt](Firewall-Rules-in-OpenWrt.md) — firewall rules required to pass VPN traffic.
- Proxy-VPNS-and-TOR — broader VPN/proxy/anonymity context.
