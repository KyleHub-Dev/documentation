---
title: Post-Installation
sidebar_position: 6
tags:
  - Initial Setup
  - Service Configuration
  - Security
  - Backup
---

## 1. Initial Pangolin Access

### 1.1 Access Pangolin Dashboard

1. Open your web browser
2. Navigate to: `https://pangolin.yourdomain.com`
3. You should see the Pangolin setup page
4. Enter the **Setup Token** that was displayed at the end of the installation (Section 3.1.6)

:::warning
If you didn't save the setup token, you can retrieve it by running this command on the Pangolin container:
```bash
cd /opt/pangolin
docker compose logs pangolin | grep "Setup Token"
```
:::

### 1.2 Complete Initial Setup

Follow the on-screen prompts to:
1. Create your admin account
2. Configure basic settings
3. Set up your organization details
4. Configure notification preferences (optional)

## 2. Adding Services to Pangolin

All services in your infrastructure should be accessed through Pangolin, not directly exposed to the internet.

### 2.1 Service Access Pattern

The recommended pattern for accessing services is:

```
Internet → Port 443 → OPNsense → Pangolin (Traefik) → Individual Services
```

This means:
- Services like Proxmox, AdGuard, NGINX Proxy Manager, etc., should **NOT** have direct WAN access
- Access them via subdomains routed through Pangolin (e.g., `proxmox.yourdomain.com`, `adguard.yourdomain.com`)
- Pangolin's Traefik reverse proxy will handle SSL termination and routing

### 2.2 Adding a Service to Pangolin

To add a service (e.g., Proxmox web interface):

1. In Pangolin dashboard, navigate to **Services** or **Containers**
2. Click **Add Service**
3. Configure the service:
   - **Name**: Proxmox
   - **URL/Address**: `https://192.168.1.2:8006` (internal Proxmox IP)
   - **Subdomain**: `proxmox` (will create `proxmox.yourdomain.com`)
   - **SSL**: Let Traefik handle SSL
4. Save the configuration
5. Pangolin will automatically:
   - Request SSL certificate from Let's Encrypt
   - Configure Traefik routing
   - Make service accessible via `https://proxmox.yourdomain.com`

### 2.3 Connecting Servers to Pangolin

To monitor and manage servers with Pangolin, install the Pangolin agent on each target server:

1. In Pangolin dashboard, navigate to **Servers**
2. Click **Add Server**
3. Copy the provided installation command, which will look similar to:

```bash
curl -fsSL https://get.digpangolin.com/agent.sh | bash -s -- \
  --url=https://pangolin.yourdomain.com \
  --token=<your-unique-token>
```

4. SSH into the target server and run the command
5. The agent will install and automatically connect to your Pangolin instance
6. Verify the connection in the Pangolin dashboard under **Servers**

:::tip
For Proxmox host and LXC containers, you may need to install the agent separately on each system you want to monitor.
:::

## 3. Security Best Practices

### 3.1 Access Control

- **Strong Authentication**: Use strong, unique passwords (minimum 16 characters)
- **API Tokens**: Regularly rotate API tokens and limit their scope
- **Session Management**: Configure appropriate session timeouts in Pangolin

### 3.2 Network Security

- **Minimize Exposure**: Only ports 80 and 443 should be accessible from WAN
- **Internal Access**: All other services should only be accessible through Pangolin or via VPN
- **Monitoring**: Regularly review access logs in Pangolin and OPNsense

### 3.3 SSL/TLS Configuration

- Pangolin's Traefik automatically handles SSL certificates via Let's Encrypt
- Certificates auto-renew before expiration
- Monitor certificate status in Pangolin dashboard

### 3.4 Regular Updates

Keep Pangolin and its dependencies up to date:

```bash
# Update Pangolin (run inside Pangolin container)
cd /opt/pangolin
docker compose pull
docker compose up -d

# Update system packages
apt update && apt upgrade -y
```