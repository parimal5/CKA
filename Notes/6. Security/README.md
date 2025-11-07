<div align="center">
  <h1><strong>CKA Exam Notes</strong></h1>
  <h3>Certified Kubernetes Administrator Study Guide</h3>
</div>

---

## 📚 Table of Contents

### Topic Overview

| Authentication                                                                          | Authorization                                                                      |
| --------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| [Certificate Basics](./Authentication/README.md#certificate-basics)                     | [Authorization Modes](./Authorization/README.md#authorization-modes)               |
| [Kubernetes mTLS Architecture](./Authentication/README.md#kubernetes-mtls-architecture) | [RBAC Components Overview](./Authorization/README.md#rbac-components-overview)     |
| [Kubernetes Certificate API](./Authentication/README.md#kubernetes-certificate-api)     | [Service Account Operations](./Authorization/README.md#service-account-operations) |
| [KubeConfig](./Authentication/README.md#kubeconfig)                                     | [Image Security](./Authorization/README.md#image-security)                         |
|                                                                                         | [Security Context](./Authorization/README.md#security-context)                     |
|                                                                                         | [NetworkPolicy](./Authorization/README.md#networkpolicy)                           |
|                                                                                         | [Custom Resource Definition](./Authorization/README.md#custom-resource-definition) |
|                                                                                         | [Custom Resource](./Authorization/README.md#custom-resource)                       |

### Core Security Concepts

Kubernetes security is built on two fundamental pillars that control cluster access and permissions:

## **🔐 Authentication**

_Who can access the cluster?_

Authentication in Kubernetes involves verifying the identity of users and services attempting to access the cluster. This includes various methods such as certificates, tokens, service accounts, and integration with external identity providers.

**[📖 Read Authentication Guide](./Authentication/README.md)**

## **🛡️ Authorization**

_What can they do once authenticated?_

Once authenticated, authorization determines what actions users and services can perform within the cluster. This covers RBAC (Role-Based Access Control), ABAC, and other authorization mechanisms that enforce security policies.

**[📖 Read Authorization Guide](./Authorization/README.md)**

## 🚀 Quick Start

1. Start with **Authentication** to understand how users and services gain access to your cluster
2. Move on to **Authorization** to learn how to control what authenticated entities can do
3. Practice with hands-on exercises in each section

---

<div align="center">
  <em>Good luck with your CKA certification journey! 🎯</em>
</div>
