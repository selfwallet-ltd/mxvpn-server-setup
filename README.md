# MxVPN Server Setup Guide

This guide explains how to set up a server for MxVPN, an SSH-based VPN application for Android.

> **Note**: This guide is intended for Debian 11+ servers.

## Disclaimer

- This guide provides **minimal setup instructions** only. Additional security measures are your responsibility.
- SelfWallet LTD provides this information "as is" without any warranty or support.
- SSH protocol does not support UDP traffic, meaning some applications requiring UDP may not work properly.
- You are responsible for securing your own server and complying with all applicable laws and regulations.

## Prerequisites

- A server running Debian 11 or newer
- Root access to the server
- Basic knowledge of Linux command line

## Installation Steps

### 1. Update Your System and Install SSH

First, make sure your system is up to date and has SSH server installed:

```bash
apt update
apt upgrade -y
apt install ssh -y
```

### 2. Configure SSH Server

Edit the SSH server configuration file:

```bash
nano /etc/ssh/sshd_config
```

Add or modify the following lines:

```
# Allow TCP forwarding
AllowTcpForwarding yes

# Allow tunneling
PermitTunnel yes

# Allow SOCKS proxy on any port
PermitOpen any
```

Save the file and restart the SSH service:

```bash
systemctl restart sshd
```

### 3. Enable IP Forwarding

Edit the sysctl configuration:

```bash
nano /etc/sysctl.conf
```

Add or uncomment these lines:

```
net.ipv4.ip_forward=1
net.ipv6.conf.all.forwarding=1
```

Apply the changes:

```bash
sysctl -p
```

### 4. Configure Firewall (iptables)

Set up iptables rules to allow forwarding and perform NAT:

```bash
# Allow forwarding for tunnel interfaces
iptables -A FORWARD -i tun+ -j ACCEPT
iptables -A FORWARD -o tun+ -j ACCEPT

# Allow incoming traffic on tunnel interfaces
iptables -A INPUT -i tun+ -j ACCEPT

# Set up NAT for outgoing traffic
# Replace ens18 with your external network interface name
iptables -t nat -A POSTROUTING -o ens18 -j MASQUERADE
```

To make these rules persistent across reboots, you should save them to a script that runs at startup or use your distribution's firewall configuration tools.

### 5. Create a User (Optional)

For better security, it's recommended to create a dedicated user for VPN connections:

```bash
adduser vpnuser
```

Follow the prompts to set a secure password.

## Connecting to Your Server

### Using MxVPN

#### Option 1: Password Authentication

Create a `.mxvpn` file with the following content:

```
[Peer]
Login = username
Password = YourStrongPassword
Endpoint = your-server-ip:22
```

#### Option 2: SSH Key Authentication

Generate an SSH key pair if you don't already have one:

```bash
ssh-keygen -t rsa -b 4096
```

Add the public key to the authorized_keys file on your server:

```bash
ssh-copy-id username@your-server-ip
```

Then create a `.mxvpn` file with the following content:

```
[Peer]
Login = username
Endpoint = your-server-ip:22
-----BEGIN OPENSSH PRIVATE KEY-----
Your private key content goes here...
-----END OPENSSH PRIVATE KEY-----
```

## Additional Security Considerations

1. **Change SSH Port**: Edit `/etc/ssh/sshd_config` to use a non-standard port
2. **Disable Password Authentication**: Set `PasswordAuthentication no` for key-only access
3. **Implement Fail2ban**: Install `fail2ban` to protect against brute force attacks
4. **Use UFW**: Consider using UFW as a simpler alternative to iptables
5. **Regular Updates**: Keep your system updated regularly

## Limitations

- **No UDP Support**: SSH tunneling does not support UDP traffic
- **Performance**: May be slower than dedicated VPN protocols like OpenVPN or WireGuard
- **App-level Split Tunneling**: Traffic can be split by apps in the MxVPN application settings

## Download MxVPN

MxVPN is available on Google Play:

[https://play.google.com/store/apps/details?id=io.selfwallet.mxvpn](https://play.google.com/store/apps/details?id=io.selfwallet.mxvpn)

---

© 2025 SelfWallet LTD. This guide is provided as-is without any warranty or support obligations.
