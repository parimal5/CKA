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

---

### mTLS Communication Requirements

Each component needs three key pieces:

1. **Certificate** (`.crt`) - Public identity signed by CA
2. **Private Key** (`.key`) - Secret key proving certificate ownership
3. **CA Certificate** (`ca.crt`) - Common trust anchor for verification

---

### How mTLS Works

1. **Component Setup**: Each component has its own cert/key pair + CA cert
2. **Communication Process**:
   - Client presents certificate to prove identity
   - Server presents certificate to prove identity
   - Both sides validate certificates against trusted CA

---

**Example: API Server ↔ etcd mTLS**\

API Server (client) → etcd (server):

API Server uses:

- `apiserver-etcd-client.crt` + `apiserver-etcd-client.key`

- `etcd/ca.crt` (to verify etcd server)

etcd uses:

- `etcd/server.crt` + `etcd/server.key`

- `etcd/ca.crt` (to verify API server client cert)

---

### Useful Commands

### View Certificate Details

```bash
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -noout -text
```

When Client need to autnticate to server it will present its own certificates and key.

```
        Client                                  Server
    (client.crt,key)                      (server.crt,key)
            |                                     |
            |<------ Mutual TLS handshake ------->|
            |                                     |
                      Common CA (or trusted CAs)

```

### Key Takeaways

- All Kubernetes components use mTLS for secure communication
- Each component needs its own certificate, private key, and CA certificate
- The CA certificate serves as the common trust anchor across all components
- File naming conventions help identify certificate types at a glance

## Kubernetes Certificate API

### The Problem Scenario

Imagine you're a Kubernetes cluster admin, and a new team member (Parimal) joins your organization. They need cluster access, so traditionally you'd have to:

1. Have them generate a private key and Certificate Signing Request (CSR)
2. Manually sign their certificate using the cluster's Certificate Authority (CA)
3. Send the signed certificate back to them
4. Repeat this process every time certificates expire or new members join

This manual process becomes tedious and error-prone as your team grows.

### The Solution: Kubernetes Certificate API

Kubernetes provides the **Certificate API** to automate and streamline this certificate management process through the `CertificateSigningRequest` resource.

---

### Step-by-Step Implementation

#### 1. User Generates Private Key and CSR (Same as Before)

The new admin still creates their credentials locally:

```bash
# Generate private key
openssl genrsa -out parimal.key 3072

# Create certificate signing request
openssl req -new -key parimal.key -out parimal.csr -subj "/CN=parimal"
```

#### 2. Prepare CSR for Kubernetes

Convert the CSR to base64 format (required by Kubernetes):

```bash
cat parimal.csr | base64 | tr -d "\n"
```

#### 3. Create CertificateSigningRequest Object

Submit the CSR to the Kubernetes cluster:

```yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: parimal-csr
spec:
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1Zq...... # Base64 encoded CSR
  signerName: kubernetes.io/kube-apiserver-client
  usages:
    - client auth
  expirationSeconds: 86400 # Optional: 24 hours (default: 1 year)
```

Apply the resource:

```bash
kubectl apply -f parimal-csr.yaml
```

#### 4. Admin Reviews and Approves

Cluster admins can now manage certificate requests through kubectl:

```bash
# List all pending CSRs
kubectl get csr

# View specific CSR details
kubectl get csr parimal-csr -o yaml

# Approve the request
kubectl certificate approve parimal-csr

# Or deny if needed
kubectl certificate deny parimal-csr
```

#### 5. Extract the Signed Certificate

Once approved, extract the signed certificate:

```bash
kubectl get csr parimal-csr -o jsonpath='{.status.certificate}' | base64 -d > parimal.crt
```

---

### Important Notes

- **signerName**: `kubernetes.io/kube-apiserver-client` is for client certificates that authenticate to the API server
- **usages**: Defines how the certificate can be used (`client auth` for API client authentication)

### Common Signer Names

- `kubernetes.io/kube-apiserver-client` - For API server client certificates
- `kubernetes.io/kube-apiserver-client-kubelet` - For kubelet client certificates
- `kubernetes.io/kubelet-serving` - For kubelet serving certificates

---

### Additional Resources

- [Official K8s Documentation: Certificate Signing Requests](https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/)
- [Managing TLS Certificates in a Cluster](https://kubernetes.io/docs/tasks/tls/managing-tls-in-a-cluster/)
- [Certificate Signing Request Tutorial](https://kubernetes.io/docs/tasks/tls/certificate-issue-client-csr/)
