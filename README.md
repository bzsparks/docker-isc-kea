# 🌐 docker-isc-kea

![ISC Kea](https://img.shields.io/badge/ISC_Kea-DHCP-0078D4?style=flat-square&logo=data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAA4AAAAOCAYAAAAfSC3RAAAABHNCSVQICAgIfAhkiAAAAAlwSFlzAAAA7AAAAOwBeShxvQAAABl0RVh0U29mdHdhcmUAd3d3Lmlua3NjYXBlLm9yZ5vuPBoAAAD9SURBVCiRhZK9SgNBFIW/2U02m2yCQohYWFnYWdvY+AY+gI+QRrC2EuxEbHwBn8DCQrAQBBshQSS7yW5mZ+7MXYsEE0X0wIU7c+655y/inKO/XwKAiKCq3N3dcXNzw/X1NRcXF5yfn3N2dsbp6SknJyccHx9zdHTEwcEBe3t77O7usrOzw/b2NltbW2xubpIkyf8mVVVms1kYhqFzzlFVFVEUEccxSZKQpim73S55ntM0DUVRUJYlZVkSRRG+7+N5Hp7n4bouu90Oz/NwXRfHcbBtG8uysCwL0zQxDANd18myDNu2CYKAIDB5fHwEoG1bXl5eeHp64vn5maenJ+r6l+fnZ+r6B7fXU/wAzR5YAAAAAElFTkSuQmCC)
![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?style=flat-square&logo=docker&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green?style=flat-square)

> **Production-ready, high-availability ISC Kea DHCPv4 deployment with TLS-secured HA channel, Dynamic DNS integration, and HTTPS API access via Caddy reverse proxy.**

---

## 📋 Overview

This repository provides a **fully Dockerized ISC Kea DHCPv4 infrastructure** designed for production environments requiring high availability and security. The solution implements a **hot-standby HA configuration** with automatic failover between primary and secondary servers, ensuring uninterrupted DHCP services.

Key architectural highlights include:
- **TLS-encrypted HA communication channel** using Ed25519 certificates with mutual authentication
- **HTTPS-secured control API** via Caddy reverse proxy with Let's Encrypt automatic certificate management
- **Dynamic DNS (DDNS)** updates with TSIG authentication for secure DNS record management
- **Modular subnet configuration** supporting easy expansion and management
- **Separate primary/secondary stacks** for independent deployment and testing

---

## ✨ Features

- 🔄 **Hot-Standby High Availability** — Automatic failover with configurable heartbeat monitoring
- 🔒 **TLS-Encrypted HA Channel** — Mutual certificate authentication between DHCP servers
- 🌐 **HTTPS Control API** — Secure external access via Caddy with Let's Encrypt certificates
- 📡 **Dynamic DNS Integration** — TSIG-secured DNS updates for forward and reverse zones
- 🧩 **Modular Subnet Configuration** — JSON-based subnet includes for easy management
- 🐳 **Docker-Based Deployment** — Reproducible infrastructure with Docker Compose
- 🧭 **Independent Primary/Secondary** — Separate configurations for flexible deployment
- ⚡ **Ed25519 Certificates** — Modern, performant cryptography for inter-server communication

---

## 🏗️ Architecture

### System Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                         DHCP Clients (UDP 67)                       │
└────────────┬───────────────────────────────────────┬────────────────┘
             │                                       │
             ▼                                       ▼
    ┌────────────────────┐                 ┌────────────────────┐
    │   PRIMARY HOST     │◄────mTLS HA────►│  SECONDARY HOST    │
    │                    │    (port 8001)  │                    │
    │  ┌──────────────┐  │                 │  ┌──────────────┐  │
    │  │ Kea DHCPv4   │  │                 │  │ Kea DHCPv4   │  │
    │  │  (Primary)   │  │                 │  │ (Secondary)  │  │
    │  └──────┬───────┘  │                 │  └──────┬───────┘  │
    │         │          │                 │         │          │
    │  ┌──────▼───────┐  │                 │  ┌──────▼───────┐  │
    │  │ Kea DDNS     │──┼─────────────────┼─►│ Kea DDNS     │  │
    │  │  Server      │  │   DDNS Updates  │  │  Server      │  │
    │  └──────────────┘  │   (unencrypted) │  └──────────────┘  │
    │         │          │    internal     │         │          │
    │  ┌──────▼───────┐  │                 │  ┌──────▼───────┐  │
    │  │   Caddy      │  │                 │  │   Caddy      │  │
    │  │ Reverse Proxy│  │                 │  │ Reverse Proxy│  │
    │  └──────┬───────┘  │                 │  └──────┬───────┘  │
    └─────────┼──────────┘                 └─────────┼──────────┘
              │ HTTPS (443)                          │ HTTPS (443)
              │ Let's Encrypt                        │ Let's Encrypt
              ▼                                      ▼
         External API Access                   External API Access
              │                                      │
              └──────────────────┬───────────────────┘
                                 ▼
                        DNS Servers (TSIG)
                        Forward/Reverse Zones
```

### Component Overview

| Component | Role | Ports | Security | Notes |
|-----------|------|-------|----------|-------|
| **Kea DHCPv4** | DHCP lease management | UDP 67 (DHCP)<br>TCP 8000 (Control API)<br>TCP 8001 (HA) | mTLS for HA channel<br>HTTPS for Control API | Hot-standby mode with automatic failover |
| **Kea DDNS** | Dynamic DNS updates | TCP 53001 (internal) | TSIG authentication | Unencrypted internal comms, bound to Docker bridge |
| **Caddy** | Reverse proxy | TCP 443 (HTTPS) | Let's Encrypt TLS<br>Cloudflare DNS-01 challenge | Automatic certificate renewal |
| **Docker Bridge** | Container networking | Custom subnet | Isolated internal network | Shared `kea-data` volume for configs |

---

## 📦 Prerequisites

Before deploying, ensure you have:

- **Docker Engine** 20.10+ and **Docker Compose** v2.0+
- **Public DNS domain(s)** for API endpoints (e.g., `kea-primary.example.com`, `kea-secondary.example.com`)
- **Cloudflare API Token** with the following scopes:
  - `Zone:DNS:Edit` for your domain
  - `Zone:Zone:Read` for zone information
- **Two hosts** (physical/virtual) or one host with isolated networks for primary/secondary stacks
- **OpenSSL 1.1.1+** for Ed25519 key/certificate generation
- **Firewall rules** allowing:
  - UDP 67 (DHCP) from client networks
  - TCP 443 (HTTPS) for external API access
  - TCP 8001 (HA channel) between primary and secondary (if on separate hosts)

---

## 📂 Directory Structure

```
.
├── primary/
│   ├── docker-compose.yaml          # Primary stack definition
│   ├── dockerfile-caddy/
│   │   └── Dockerfile               # Caddy with Cloudflare DNS plugin
│   ├── config/
│   │   ├── kea-dhcp4.conf           # DHCPv4 configuration (primary)
│   │   └── kea-dhcp-ddns.conf       # DDNS configuration
│   └── leases/                      # Persistent lease database
├── secondary/
│   ├── docker-compose.yaml          # Secondary stack definition
│   ├── dockerfile-caddy/
│   │   └── Dockerfile
│   ├── config/
│   │   ├── kea-dhcp4.conf           # DHCPv4 configuration (secondary)
│   │   └── kea-dhcp-ddns.conf
│   └── leases/
├── kea-data/                        # Shared configuration data
│   ├── Caddyfile                    # Caddy reverse proxy config
│   ├── ha-peers-primary.json        # HA peer definitions (primary)
│   ├── ha-peers-secondary.json      # HA peer definitions (secondary)
│   ├── subnet-id1.json              # Subnet configurations (modular)
│   ├── subnet-id2.json
│   ├── subnet-id3.json
│   ├── forward-ddns.json            # Forward DNS zone config
│   ├── reverse-ddns.json            # Reverse DNS zone config
│   ├── tsig-keys.json               # TSIG authentication keys
│   ├── server.crt                   # TLS certificate (HA channel)
│   └── server.key                   # TLS private key (HA channel)
├── .gitignore
└── README.md
```

---

## 🔐 TLS Certificate Setup

<details>
<summary><strong>Why Ed25519?</strong></summary>

Ed25519 offers significant advantages for the HA communication channel:
- **Performance**: Faster key generation and signature verification
- **Security**: 128-bit security level equivalent to RSA 3072-bit
- **Key Size**: Compact 256-bit keys reduce storage and transmission overhead
- **Modern Support**: Widely supported in OpenSSL 1.1.1+ and Kea 2.0+

</details>

### Current Simplified Approach (Self-Signed)

The current implementation uses a **single self-signed certificate** that acts as both the CA and server certificate. This simplifies initial deployment but has limitations for production environments.

#### Generate Ed25519 Private Key

```bash
openssl genpkey -algorithm ed25519 -out server.key
```

#### Generate Certificate Signing Request (CSR)

```bash
openssl req -new -subj "/CN=kea-ha-primary.bzsprks.com" -key server.key -out server.csr \
  -addext "subjectAltName = DNS:kea-ha-primary.bzsprks.com,DNS:kea-ha-secondary.bzsprks.com"
```

> **Note**: Update the domain names (`bzsprks.com`) to match your environment.

#### Generate 10-Year Self-Signed Certificate

```bash
openssl x509 -req -sha256 -extfile my.cnf -extensions ext \
  -in server.csr -days 3650 -signkey server.key -out server.crt
```

Create a `my.cnf` file with Subject Alternative Names:

```ini
[ext]
subjectAltName = DNS:kea-ha-primary.bzsprks.com,DNS:kea-ha-secondary.bzsprks.com
keyUsage = critical, digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth, clientAuth
```

#### Deploy Certificates

```bash
# Set secure permissions
chmod 600 kea-data/server.key
chmod 644 kea-data/server.crt

# In Kea DHCP config, specify server.crt as BOTH the CA and server certificate
# This works because the certificate is self-signed
```

### Recommended Production Approach (Proper CA)

<details>
<summary><strong>Click to expand: Production-grade PKI setup</strong></summary>

For enhanced security, create a dedicated Certificate Authority and issue separate certificates for each node:

#### 1. Create Certificate Authority

```bash
# Generate CA private key
openssl genpkey -algorithm ed25519 -out ca.key
chmod 600 ca.key

# Create CA certificate (10 years)
openssl req -new -x509 -key ca.key -out ca.crt -days 3650 \
  -subj "/CN=Kea HA Certificate Authority/O=YourOrg"
```

#### 2. Generate Primary Server Certificate

```bash
# Private key
openssl genpkey -algorithm ed25519 -out primary.key
chmod 600 primary.key

# CSR with Subject Alternative Name
openssl req -new -key primary.key -out primary.csr \
  -subj "/CN=kea-ha-primary.example.com" \
  -addext "subjectAltName = DNS:kea-ha-primary.example.com,IP:10.0.1.10"

# Create config for signing
cat > primary-ext.cnf <<EOF
subjectAltName = DNS:kea-ha-primary.example.com,IP:10.0.1.10
keyUsage = critical, digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth, clientAuth
EOF

# Sign with CA
openssl x509 -req -in primary.csr -CA ca.crt -CAkey ca.key \
  -CAcreateserial -out primary.crt -days 365 -sha256 \
  -extfile primary-ext.cnf
```

#### 3. Generate Secondary Server Certificate

```bash
# Private key
openssl genpkey -algorithm ed25519 -out secondary.key
chmod 600 secondary.key

# CSR
openssl req -new -key secondary.key -out secondary.csr \
  -subj "/CN=kea-ha-secondary.example.com" \
  -addext "subjectAltName = DNS:kea-ha-secondary.example.com,IP:10.0.1.11"

# Sign with CA
cat > secondary-ext.cnf <<EOF
subjectAltName = DNS:kea-ha-secondary.example.com,IP:10.0.1.11
keyUsage = critical, digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth, clientAuth
EOF

openssl x509 -req -in secondary.csr -CA ca.crt -CAkey ca.key \
  -CAcreateserial -out secondary.crt -days 365 -sha256 \
  -extfile secondary-ext.cnf
```

#### 4. Verify Certificates

```bash
# Verify primary certificate
openssl verify -CAfile ca.crt primary.crt

# Verify secondary certificate
openssl verify -CAfile ca.crt secondary.crt

# Check certificate details
openssl x509 -in primary.crt -noout -text
```

#### 5. Configure Kea for mTLS

Update `kea-dhcp4.conf` on the **primary**:

```json
"high-availability": [{
    "this-server-name": "kea-dhcp4-primary",
    "trust-anchor": "/etc/kea/ca.crt",
    "cert-file": "/etc/kea/primary.crt",
    "key-file": "/etc/kea/primary.key",
    "require-client-certs": true,
    ...
}]
```

Update `kea-dhcp4.conf` on the **secondary**:

```json
"high-availability": [{
    "this-server-name": "kea-dhcp4-secondary",
    "trust-anchor": "/etc/kea/ca.crt",
    "cert-file": "/etc/kea/secondary.crt",
    "key-file": "/etc/kea/secondary.key",
    "require-client-certs": true,
    ...
}]
```

#### 6. Certificate Rotation

For annual certificate rotation:

```bash
# Renew certificates (repeat steps 2-3 with new dates)
# Deploy new certificates
cp new-primary.crt kea-data/primary.crt
cp new-primary.key kea-data/primary.key

# Restart Kea containers
docker compose -f primary/docker-compose.yaml restart kea-dhcp4-primary
```

</details>

### Security Architecture Summary

| Channel | Encryption | Notes |
|---------|-----------|-------|
| **HA Communication** (port 8001) | mTLS with Ed25519 | Self-signed 10-year cert or CA-issued with annual rotation |
| **Control API** (port 443) | HTTPS via Caddy | Let's Encrypt with automatic 90-day renewal |
| **DDNS Updates** (internal) | None | Unencrypted on Docker bridge; secured with TSIG to upstream DNS |

> **Note**: In the Kea DHCP config, when using the simplified approach, `server.crt` is specified as **both** the CA (`trust-anchor`) and server certificate (`cert-file`) since it is self-signed.

---

## ⚙️ Configuration

### Key Configuration Files

#### `kea-dhcp4.conf`

Core DHCP server configuration:

```json
{
  "Dhcp4": {
    "interfaces-config": {
      "interfaces": ["eth0"]
    },
    "hooks-libraries": [{
      "library": "/usr/lib/kea/hooks/libdhcp_ha.so",
      "parameters": {
        "high-availability": [{
          "this-server-name": "kea-dhcp4-primary",
          "mode": "hot-standby",
          "trust-anchor": "/etc/kea/server.crt",
          "cert-file": "/etc/kea/server.crt",
          "key-file": "/etc/kea/server.key",
          "require-client-certs": true,
          "peers": [
            <?include "/etc/kea/ha-peers-primary.json"?>
          ]
        }]
      }
    }],
    "subnet4": [
      <?include "/etc/kea/subnet-id1.json"?>,
      <?include "/etc/kea/subnet-id2.json"?>
    ]
  }
}
```

**Key settings**:
- `mode: hot-standby` — One active server, one standby
- `require-client-certs: true` — Mutual TLS authentication
- Modular subnet includes for easy management

#### `ha-peers-primary.json`

HA peer definitions for the primary server:

```json
{
  "name": "kea-dhcp4-secondary",
  "url": "https://kea-ha-secondary.example.com:8001/",
  "role": "standby",
  "auto-failover": true
}
```

#### `kea-dhcp-ddns.conf`

Dynamic DNS configuration:

```json
{
  "DhcpDdns": {
    "ip-address": "192.168.252.3",
    "port": 53001,
    <?include "/etc/kea/tsig-keys.json"?>,
    "forward-ddns": {
      <?include "/etc/kea/forward-ddns.json"?>
    },
    "reverse-ddns": {
      <?include "/etc/kea/reverse-ddns.json"?>
    }
  }
}
```

#### Example Subnet Configuration

Create modular subnet files in `kea-data/`:

```json
{
  "subnet": "10.0.10.0/24",
  "id": 1,
  "pools": [
    { "pool": "10.0.10.100 - 10.0.10.200" }
  ],
  "option-data": [
    { "name": "routers", "data": "10.0.10.1" },
    { "name": "domain-name-servers", "data": "10.10.10.10, 10.10.10.11" }
  ],
  "reservations": [
    {
      "hw-address": "aa:bb:cc:dd:ee:ff",
      "ip-address": "10.0.10.50",
      "hostname": "printer01"
    }
  ]
}
```

#### `Caddyfile`

Caddy reverse proxy configuration:

```caddyfile
kea-primary.example.com {
    reverse_proxy kea-dhcp4-primary:8000
    tls {
        dns cloudflare {env.CLOUDFLARE_API_TOKEN}
    }
}
```

---

## 🚀 Deployment

### Quick Start

#### 1. Clone and Configure Environment

```bash
git clone https://github.com/bzsparks/docker-isc-kea.git
cd docker-isc-kea

# Create environment files for primary and secondary
cat > primary/.env <<EOF
KEA_VERSION=2.6.2
CADDY_SERVER_IP=192.168.252.2
DHCPv4_SERVER_IP=192.168.252.10
DDNS_SERVER_IP=192.168.252.3
NETWORK_CIDR=192.168.252.0/24
MY_DOMAIN=kea-primary.example.com
CLOUDFLARE_API_TOKEN=your_cloudflare_token_here
EOF

cp primary/.env secondary/.env
# Edit secondary/.env to change MY_DOMAIN to kea-secondary.example.com
```

#### 2. Generate TLS Certificates

```bash
# Create certificates directory
mkdir -p kea-data

# Generate certificates (using simplified self-signed approach)
openssl genpkey -algorithm ed25519 -out kea-data/server.key
openssl req -new -subj "/CN=kea-ha-primary.example.com" \
  -key kea-data/server.key -out kea-data/server.csr \
  -addext "subjectAltName = DNS:kea-ha-primary.example.com,DNS:kea-ha-secondary.example.com"

# Create OpenSSL config
cat > kea-data/cert-ext.cnf <<EOF
[ext]
subjectAltName = DNS:kea-ha-primary.example.com,DNS:kea-ha-secondary.example.com
keyUsage = critical, digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth, clientAuth
EOF

openssl x509 -req -sha256 -extfile kea-data/cert-ext.cnf -extensions ext \
  -in kea-data/server.csr -days 3650 -signkey kea-data/server.key \
  -out kea-data/server.crt

# Set permissions
chmod 600 kea-data/server.key
chmod 644 kea-data/server.crt
```

#### 3. Deploy Primary Stack

```bash
cd primary
docker compose up -d

# Verify services are running
docker compose ps
docker compose logs -f
```

#### 4. Deploy Secondary Stack

```bash
cd ../secondary
docker compose up -d

# Check HA status
docker compose logs kea-dhcp4-secondary | grep "high-availability"
```

#### 5. Verify Deployment

```bash
# Check HA status via API (primary)
curl https://kea-primary.example.com/ha-heartbeat

# Get lease statistics
curl https://kea-primary.example.com/stat-lease4-get

# Check DDNS service
docker exec kea-ddns-primary cat /var/log/kea-dhcp-ddns.log
```

#### 6. Test DHCP Lease Assignment

```bash
# On a test client in the DHCP network
sudo dhclient -v eth0

# Verify lease was issued and DNS updated
dig +short test-client.example.com
```

---

## 🔒 Security Considerations

### Multi-Layer Security Architecture

1. **HA Communication Channel**
   - mTLS with mutual certificate authentication
   - Restrict to Docker bridge network or host firewall allowlist
   - Use strong Ed25519 cryptography
   - Regular certificate rotation (recommended annually for CA-issued certs)

2. **Control API Access**
   - TLS termination at Caddy with Let's Encrypt
   - Consider additional authentication:
     - HTTP Basic Auth for simple protection
     - Client certificates for mTLS
     - IP allowlisting via Caddy configuration
   - API exposed only on port 443 (HTTPS)

3. **DDNS Security**
   - Unencrypted communications on **Docker bridge only** (never exposed externally)
   - TSIG authentication for upstream DNS updates
   - Egress-only traffic from DDNS containers

4. **Secrets Management**
   - Never commit `.key`, `.env`, or `tsig-keys.json` to version control (see `.gitignore`)
   - Set restrictive permissions: `600` for private keys, `640` for configs with secrets
   - Consider Docker secrets or external secret management for production

5. **Operational Security**
   - Pin Docker image digests for reproducible deployments
   - Use least-privilege Cloudflare API tokens (Zone:DNS:Edit + Zone:Zone:Read only)
   - Regularly update Kea and Caddy images for security patches
   - Monitor logs for unauthorized access attempts
   - Disable unused control socket endpoints

---

## 🔧 Troubleshooting

<details>
<summary><strong>Common Issues and Solutions</strong></summary>

### Let's Encrypt Certificate Failures

**Symptoms**: Caddy fails to obtain certificates, "challenge failed" errors

**Solutions**:
```bash
# Check DNS propagation
dig +short _acme-challenge.kea-primary.example.com TXT

# Verify Cloudflare API token scopes
curl -X GET "https://api.cloudflare.com/client/v4/user/tokens/verify" \
  -H "Authorization: Bearer $CLOUDFLARE_API_TOKEN"

# Check Caddy logs
docker compose logs caddy | grep -i acme
```

### HA TLS Handshake Errors

**Symptoms**: "TLS handshake failed", "certificate verify failed", HA not synchronizing

**Solutions**:
```bash
# Verify certificate CN/SAN matches hostname
openssl x509 -in kea-data/server.crt -noout -text | grep -A1 "Subject Alternative Name"

# Check certificate trust chain
openssl verify -CAfile kea-data/server.crt kea-data/server.crt

# Verify system clocks are synchronized (clock skew > 5 minutes causes TLS failures)
timedatectl status

# Check Kea HA logs
docker compose logs kea-dhcp4-primary | grep -i "tls\|handshake"
```

### DDNS Update Failures

**Symptoms**: DNS records not updating, TSIG errors, SERVFAIL responses

**Solutions**:
```bash
# Test TSIG key manually
nsupdate -k /path/to/tsig.key <<EOF
server 10.10.10.10
zone example.com
update add test.example.com 300 A 10.0.10.100
send
EOF

# Check DDNS logs
docker exec kea-ddns-primary tail -f /var/log/kea-dhcp-ddns.log

# Verify DNS server allows TSIG-signed updates
dig @10.10.10.10 +tcp example.com SOA
```

### DHCP Not Listening on Interface

**Symptoms**: Clients not receiving DHCP offers, "failed to bind socket" errors

**Solutions**:
```bash
# Verify container has required capabilities
docker inspect kea-dhcp4-primary | grep -A10 CapAdd
# Should include: NET_ADMIN, NET_BIND_SERVICE, NET_RAW

# Check interface configuration
docker exec kea-dhcp4-primary ip addr show eth0

# Verify no other DHCP server is listening
sudo netstat -ulnp | grep :67
```

### Container Health Checks

```bash
# Check container health status
docker compose ps

# View detailed logs
docker compose logs --tail=100 kea-dhcp4-primary

# Execute interactive shell for debugging
docker exec -it kea-dhcp4-primary /bin/bash

# Test Kea control API internally
docker exec kea-dhcp4-primary curl -s http://localhost:8000/version
```

</details>

---

## 📚 References

### ISC Kea Documentation

- [Kea Administrator Reference Manual (ARM)](https://kea.readthedocs.io/en/latest/arm/)
- [High Availability Hook](https://kea.readthedocs.io/en/latest/arm/hooks.html#ha-high-availability)
- [HTTPS/TLS Support](https://kea.readthedocs.io/en/kea-3.0.2/arm/hooks.html#https-support)
- [DHCPv4 Configuration](https://kea.readthedocs.io/en/latest/arm/dhcp4-srv.html)
- [DDNS Integration](https://kea.readthedocs.io/en/latest/arm/ddns.html)
- [Control Channel](https://kea.readthedocs.io/en/latest/arm/ctrl-channel.html)

### Caddy Documentation

- [Reverse Proxy Guide](https://caddyserver.com/docs/caddyfile/directives/reverse_proxy)
- [Automatic HTTPS](https://caddyserver.com/docs/automatic-https)
- [Cloudflare DNS Plugin](https://github.com/caddy-dns/cloudflare)

### Related Resources

- [Docker Documentation](https://docs.docker.com/)
- [Docker Compose Reference](https://docs.docker.com/compose/compose-file/)
- [OpenSSL Ed25519 Keys](https://www.openssl.org/docs/man1.1.1/man1/genpkey.html)
- [RFC 8032: Edwards-Curve Digital Signature Algorithm (EdDSA)](https://www.rfc-editor.org/rfc/rfc8032)
- [RFC 2845: TSIG (Secret Key Transaction Authentication)](https://www.rfc-editor.org/rfc/rfc2845)

---

## 📄 License

This project is licensed under the **MIT License**.

---

## 🤝 Contributing

Contributions are welcome! Please feel free to submit issues or pull requests.

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

---

**Built with ❤️ for reliable, secure DHCP infrastructure**











