# TLS Security Interview Questions and Answers

This document contains common interview questions related to TLS (Transport Layer Security), Certificate Management, and securing infrastructure.

### 1. What is TLS?
**Answer:**
**TLS (Transport Layer Security)** is a protocol that encrypts communication between a client and a server.

**It ensures:**
*   **Confidentiality:** Data cannot be read by attackers.
*   **Integrity:** Data cannot be modified in transit.
*   **Authentication:** Server identity is verified.

**Example:** When you access `https://example.com`, TLS encrypts the connection between your browser and the server.

### 2. What happens during a TLS handshake?
**Answer:**
1.  **Client Hello:** Client sends supported TLS versions and cipher suites.
2.  **Server Hello:** Server sends its certificate.
3.  **Verification:** Client verifies the certificate (CA trust + domain match).
4.  **Key Exchange:** Client and server agree on an encryption key.
5.  **Secure Session:** Encrypted communication begins.

**Important:** The certificate proves the server’s identity.

### 3. What is a Certificate Authority (CA)?
**Answer:**
A **Certificate Authority** is a trusted organization that issues digital certificates. Browsers trust certificates signed by known CAs.

**In AWS:**
*   **Public certificates:** Issued via AWS Certificate Manager (ACM).
*   **Internal certificates:** Issued via AWS Private CA.


### 4. Where should TLS termination happen?
**Answer:**
TLS termination can happen at different layers depending on security requirements:

1.  **CDN Layer (CloudFront):**
    *   Reduces latency and protects the origin.
2.  **Load Balancer Layer (ALB/NLB):**
    *   **Most Common Pattern.**
    *   ALB handles the TLS handshake and decryption, then forwards HTTP internally to reduce load on backend servers.
3.  **Application Level (End-to-End):**
    *   Terminate TLS inside the application container.
    *   **Use Case:** Compliance requirements (e.g., HIPAA, PCI-DSS) where traffic must be encrypted all the way to the backend.

**Architect-level answer:** Terminate at the edge, then re-encrypt to the backend for a Zero-Trust architecture.

### 5. What is end-to-end encryption?
**Answer:**
Traffic is encrypted at every hop:
`Client → Load Balancer → Application → Database`

Even inside the private network (VPC), traffic remains encrypted.
**Prevents:**
*   Insider threats.
*   Network sniffing.
*   Lateral movement risks.

### 6. How do you manage TLS certificates in EKS?
**Answer:**
**Common Approaches:**

**1️⃣ Use AWS ACM with ALB Ingress Controller (Production Standard)**
*   Create a certificate in **AWS ACM**.
*   Deploy the **AWS Load Balancer Controller**.
*   Attach the certificate via an Ingress annotation.
*   **Result:** ALB handles TLS termination.

**2️⃣ Use cert-manager in Kubernetes**
*   **cert-manager** automates certificate creation inside the cluster.
*   Integrates with **Let’s Encrypt** or **Private CA**.
*   Handles renewal automatically.
*   **Use Case:** mTLS, internal services, custom domain routing.

### 7. What is mTLS?
**Answer:**
**Mutual TLS (mTLS)** means:
*   Server verifies the client certificate.
*   Client verifies the server certificate.

**Use Cases:**
*   Service-to-service communication.
*   Zero-trust architecture.
*   Financial / Healthcare systems.

**Implementation in EKS:**
*   Via **Service Mesh** (Istio, Linkerd).
*   Or via **cert-manager** + Private CA.

### 8. How do you rotate certificates in Kubernetes?
**Answer:**
**Best Practices:**
*   Use automated renewal tools like **cert-manager**.
*   Avoid hardcoding certificates in container images.
*   Use **Kubernetes Secrets** for storage.
*   Enable rolling restarts of pods after renewal to pick up new certs.
*   Monitor expiration dates.
*   **Never** manually upload certificates to pods.

### 9. How do you secure internal communication between pods?
**Answer:**
*   Use **mTLS** via a Service Mesh.
*   Use internal Load Balancers with TLS listeners.
*   Encrypt traffic using private certificates.
*   Use **Network Policies** for segmentation.

**Security Principle:** Do not assume internal traffic is safe.

### 10. What are common TLS mistakes in production?
**Answer:**
*   Expired certificates.
*   Wrong region for ACM certificate (e.g., creating in `ap-south-1` for CloudFront).
*   Weak cipher suites enabled.
*   Using deprecated TLS versions (1.0 / 1.1).
*   Not enforcing HTTPS (allowing HTTP fallback).
*   Publicly exposing private keys.
*   Not rotating certificates regularly.

### 11. How do you monitor certificate expiration?
**Answer:**
*   Use **CloudWatch metrics** for ACM (`DaysToExpiry`).
*   Enable expiration alarms.
*   Monitor Kubernetes secrets expiry (using tools like `x509-exporter`).
*   Add an observability dashboard for certificate validity days.

**Architect Mindset:** Certificate expiry should never cause downtime.

### 12. How would you design certificate management for an enterprise EKS cluster?
**Answer:**
**Strong Architect Design:**

1.  **Public Traffic:**
    *   TLS terminated at **ALB** using **ACM** certificates.
2.  **Internal Service Communication:**
    *   **mTLS** enforced via a Service Mesh (Istio/Linkerd).
    *   **Private CA** for internal certificates.
3.  **Automation:**
    *   **cert-manager** for automatic issuance and renewal.
    *   CI/CD integration for certificate provisioning.
4.  **Monitoring:**
    *   Expiration alerts via CloudWatch/Prometheus.
    *   Audit logs enabled.
5.  **Security:**
    *   Secrets encrypted at rest using **KMS**.
    *   Restrict RBAC access to certificate secrets.
    
### 13. Users report HTTPS working in one region but failing in another. What do you check?
**Answer:**
**Troubleshooting Approach:**
1.  **Check Certificate Region:** CloudFront certs *must* be in `us-east-1`. ALB certs must be in the *same region* as the ALB.
2.  **Verify DNS Routing:** Is Route 53 pointing to the correct regional endpoint?
3.  **Confirm Association:** Is the certificate attached to the Load Balancer listener in the failing region?
4.  **Validate Domain:** Does the domain name match the certificate's SAN (Subject Alternative Name)?
5.  **Check TLS Policy:** Are the security policies consistent across regions?

## Difference Summary

| Level | TLS Handling |
| :--- | :--- |
| **CDN** | Edge termination (CloudFront). |
| **ALB** | Common TLS termination point. |
| **App** | End-to-end encryption (highest security). |
| **EKS** | ACM + Ingress Controller OR cert-manager. |
| **Internal** | mTLS recommended (Service Mesh). |
