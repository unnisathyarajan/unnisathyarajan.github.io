---
title: "Secure SSH Access to a VM with Cloudflare ZTNA"
description: "Configure Cloudflare ZTNA for SSH access to a Linux VM using Cloudflare Tunnel, Access policies, OTP authentication, and cloudflared ProxyCommand."
date: 2026-07-01
categories: [DevOps, Security]
tags: [cloudflare, ztna, cloudflare-tunnel, ssh, linux, devops, sre]
---

# Secure SSH Access to a VM with Cloudflare ZTNA

Cloudflare ZTNA is a clean way to put SSH behind identity-aware access instead of exposing port `22` directly to the internet. This setup uses a Cloudflare Tunnel connector, a published route, and an Access policy with OTP-based login.

## Architecture

| Component         | Role                                                      |
| ----------------- | --------------------------------------------------------- |
| Cloudflare Tunnel | Connects the VM to Cloudflare without opening inbound SSH |
| Public hostname   | Provides the SSH access hostname                          |
| Access policy     | Controls who can authenticate                             |
| `cloudflared`     | Proxies local SSH traffic through Cloudflare Access       |

> Keep direct SSH exposure closed wherever possible. Treat Cloudflare Access policies as production security controls, not just UI configuration.
{: .prompt-warning }

## Create the Cloudflare Connector

Log in to Cloudflare and add a connector for the VM.

![Cloudflare connectors](/assets/img/posts/2026/2026-07-01-cloudflare-ztna-vm/1connectors.png)

Create the tunnel and complete the connector setup.

![Create tunnel](/assets/img/posts/2026/2026-07-01-cloudflare-ztna-vm/2tunnel.png)

![Tunnel setup step](/assets/img/posts/2026/2026-07-01-cloudflare-ztna-vm/3tunnel2.png)

![Tunnel connector status](/assets/img/posts/2026/2026-07-01-cloudflare-ztna-vm/4tunnel3.png)

## Publish the SSH Route

After the connector is active, publish the route for SSH access.

![Published route](/assets/img/posts/2026/2026-07-01-cloudflare-ztna-vm/5publishedroute.png)

## Connect with SSH

Use `cloudflared access ssh` as the SSH proxy command.

```bash
ssh -o ProxyCommand="cloudflared access ssh --hostname %h" root@contxxx.xxx.xx
```

For repeated access, add a host entry to your SSH config.

```bash
Host conxxx
  Hostname conxxxx
  User root
  ProxyCommand cloudflared access ssh --hostname %h
  IdentityFile ~/.ssh/id_xxxx
```

> Replace the host placeholders with your actual Cloudflare hostname and SSH alias.
{: .prompt-info }

## Add OTP-Based Access

Create an Access application for the public hostname.

![OTP access](/assets/img/posts/2026/2026-07-01-cloudflare-ztna-vm/6otp.png)

Select the option for a public hostname.

![Public hostname application](/assets/img/posts/2026/2026-07-01-cloudflare-ztna-vm/7otp2.png)

![Application configuration](/assets/img/posts/2026/2026-07-01-cloudflare-ztna-vm/8otp3.png)

## Add Access Policies

Attach an Access policy to the application. This is where you define who can request SSH access.

![Cloudflare Access policy](/assets/img/posts/2026/2026-07-01-cloudflare-ztna-vm/9otppolicy.png)

## Test the Login Flow

Once the application and policy are linked, start an SSH session. Cloudflare prompts for identity verification before the connection is allowed.

![SSH OTP prompt](/assets/img/posts/2026/2026-07-01-cloudflare-ztna-vm/10ssh.png)

Enter the email address, submit the OTP on the Cloudflare page, and the SSH session will continue.
