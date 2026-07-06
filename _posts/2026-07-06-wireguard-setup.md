---
title: "WireGuard VPN Server Setup on Linux"
description: "A short, practical guide to setting up a self-hosted WireGuard VPN server on Linux with NAT, IPv4 forwarding, systemd, and an Android client."
date: 2026-07-06
categories: [DevOps, Security]
tags: [wireguard, vpn, linux, networking, sysadmin, devops]
---

# WireGuard VPN Server Setup on Linux

This is a compact WireGuard setup for a self-hosted Linux VPN server with one Android client.

## Install WireGuard

```bash
apt-get install wireguard
```

## Generate Server Keys

Create the server private and public keys:

```bash
wg genkey | tee privatekey | wg pubkey > publickey
```

Generate a separate key pair for every client.

## Configure the WireGuard Interface

Create `/etc/wireguard/wg0.conf`:

```ini
[Interface]
PrivateKey=WHlUd9cglxxxxxxxxxxxxxg/te9BlzGA=
Address=10.0.0.1/8
SaveConfig=true
PostUp=iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE;
PostDown=iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE;
ListenPort=51820
```

Lock down the WireGuard config directory:

```bash
chmod -R 600 /etc/wireguard/
```

| Field        | Purpose                                              |
| ------------ | ---------------------------------------------------- |
| `wg0`        | Tunnel interface name                                |
| `PrivateKey` | Server private key generated earlier                 |
| `PostUp`     | Adds forwarding and NAT rules when the tunnel starts |
| `PostDown`   | Removes the same rules when the tunnel stops         |

> Replace `eth0` with the server's actual public network interface if different.
{: .prompt-warning }

## Start and Verify WireGuard

Bring up the interface:

```bash
wg-quick up wg0
```

Check WireGuard status:

```bash
wg
```

Confirm the interface exists:

```bash
ip link
```

## Enable IPv4 Forwarding

Check whether forwarding is enabled:

```bash
cat /proc/sys/net/ipv4/ip_forward
```

If it returns `0`, enable it immediately:

```bash
sysctl -w net.ipv4.ip_forward=1
```

For a permanent change, edit `/etc/sysctl.conf`, uncomment the IPv4 forwarding line, then reload sysctl:

```bash
sysctl -p
```

## Confirm Traffic Through the Tunnel

From the client, ping `8.8.8.8`, then verify traffic on the server:

```bash
tcpdump -envi wg0 host 8.8.8.8
```

## Run WireGuard with systemd

```bash
systemctl start wg-quick@wg0
systemctl status wg-quick@wg0
systemctl enable wg-quick@wg0
```

## Configure an Android WireGuard Client

Generate Android client keys:

```bash
wg genkey | sudo tee android_private.key | wg pubkey | sudo tee android_public.key
```

Key placement:

| Key                   | Used in               |
| --------------------- | --------------------- |
| `android_private.key` | Android client config |
| `android_public.key`  | Server peer config    |
| `server_public_key`   | Android client config |

### Add Android as a Server Peer

Add this peer to the server `wg0` config:

```ini
[Peer]
PublicKey=<ANDROID_PUBLIC_KEY>
AllowedIPs=10.0.0.2/32
```

Restart WireGuard:

```bash
systemctl restart wg-quick@wg0
```

### Create the Android Config

Create `android.conf`:

```ini
[Interface]
PrivateKey=<ANDROID_PRIVATE_KEY>
Address=10.0.0.2/32
DNS=1.1.1.1

[Peer]
PublicKey=<SERVER_PUBLIC_KEY>
Endpoint=<YOUR_CONTABO_PUBLIC_IP>:51820
AllowedIPs=0.0.0.0/0
PersistentKeepalive=25
```

## Generate a QR Code

Install `qrencode`:

```bash
apt install qrencode -y
```

Print the Android config as a QR code:

```bash
qrencode -t ansiutf8 < /etc/wireguard/android.conf
```
