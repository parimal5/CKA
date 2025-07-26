<div align="center">
  <h1><strong>CKA Exam Notes</strong></h1>
  <h3>Kubernetes Security</h3>
</div>

## Overview

Kubernetes security is built on two fundamental concepts:

1. **Authentication** - Who can access the cluster?
2. **Authorization** - What can they do once authenticated?

## Certificate Basics

### What is a Certificate?

A digital file containing:

- Public key
- Identity information (domain name, organization, issuer)
- Validity period

In Kubernetes TLS, the public key is embedded within the certificate, not sent separately.

### File Naming Conventions

| Type                         | Extensions           | Examples                       |
| ---------------------------- | -------------------- | ------------------------------ |
| **Public Keys/Certificates** | `*.crt`, `*.pem`     | `server.crt`, `client.pem`     |
| **Private Keys**             | `*.key`, `*-key.pem` | `server.key`, `client-key.pem` |

> **Note**: Private key files always contain "key" in their filename

## Kubernetes mTLS Architecture

Kubernetes uses **mutual TLS (mTLS)** for secure component communication.

### Certificate Types

1. **Client Certificates** - Used by components acting as clients
2. **Server Certificates** - Used by components acting as servers

### Client vs Server Roles

| Component                         | Acts as Client to | Acts as Server for                       |
| --------------------------------- | ----------------- | ---------------------------------------- |
| **API Server**                    | etcd              | kubectl, controllers, scheduler, kubelet |
| **etcd**                          | -                 | API Server (and peers in HA)             |
| **Controllers/Scheduler/Kubelet** | API Server        | -                                        |

## mTLS Communication Requirements

Each component needs three key pieces:

1. **Certificate** (`.crt`) - Public identity signed by CA
2. **Private Key** (`.key`) - Secret key proving certificate ownership
3. **CA Certificate** (`ca.crt`) - Common trust anchor for verification

### How mTLS Works

1. **Component Setup**: Each component has its own cert/key pair + CA cert
2. **Communication Process**:
   - Client presents certificate to prove identity
   - Server presents certificate to prove identity
   - Both sides validate certificates against trusted CA

**Example: API Server ↔ etcd mTLS**\

API Server (client) → etcd (server):

API Server uses:

- `apiserver-etcd-client.crt` + `apiserver-etcd-client.key`

- `etcd/ca.crt` (to verify etcd server)

etcd uses:

- `etcd/server.crt` + `etcd/server.key`

- `etcd/ca.crt` (to verify API server client cert)

## Useful Commands

### View Certificate Details

```bash
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -noout -text
```

## Key Takeaways

- All Kubernetes components use mTLS for secure communication
- Each component needs its own certificate, private key, and CA certificate
- The CA certificate serves as the common trust anchor across all components
- File naming conventions help identify certificate types at a glance
