# Transpedite v2 — Architecture (Sprint 1)

> Visual overview of the infrastructure deployed in Sprint 1. Maintained by Toucan Talent.
> Last updated: 2026-05-12

---

## High-level architecture

```mermaid
flowchart TB
    subgraph EXTERNAL["🌍 EXTERNAL"]
        direction TB
        USER["👨‍⚕️ Med-Rok Users<br/>(Hospital Admins, Case Managers,<br/>Intake Persons)"]
        INTERNET((Internet))
    end

    subgraph AWSACC["☁️ AWS Account (us-east-2, Ohio)"]
        direction TB

        subgraph EDGE["🛡️ Edge"]
            R53[🌐 Route 53<br/>DNS]
            EIP["📍 Elastic IP<br/>(stable public IP)"]
            NIPIO["🔗 nip.io<br/>(temporary DNS for Sprint 1)"]
        end

        subgraph VPC["🔵 Dedicated VPC (transpedite-vpc-dev)"]
            direction TB

            subgraph PUBSUB["📂 Public Subnets (2 Availability Zones)"]
                IGW[Internet Gateway]
                NAT[NAT Gateway]
                EC2["💻 EC2 t3.small<br/>Ubuntu 22.04 LTS<br/>EBS gp3 🔒 KMS-encrypted"]
            end

            subgraph PRIVSUB["📁 Private Subnets (2 AZs)"]
                FUT["Future: RDS, ECS<br/>(Sprint 2+)"]
            end

            SG["🛡️ Security Group<br/>Only ports 80/443 from Internet<br/>NO SSH (access via AWS SSM)"]
        end

        subgraph DOCKER["🐳 Docker Compose Stack (inside EC2)"]
            direction LR
            CADDY["🔐 Caddy 2<br/>HTTPS + Let's Encrypt<br/>HSTS + Security Headers"]
            RAILS["💎 Rails 7.2 / Ruby 3.3<br/>Puma 6<br/>Transpedite v2 app"]
            POSTGRES["🐘 PostgreSQL 16<br/>scram-sha-256<br/>3 DBs + 3 differentiated users"]
            REDIS["🔴 Redis 7<br/>auth + persistence"]
            CADDY -.->|reverse_proxy| RAILS
            RAILS -.->|TLS| POSTGRES
            RAILS -.->|auth| REDIS
        end

        subgraph SERVICES["⚙️ AWS Supporting Services"]
            direction LR
            subgraph S3["🪣 S3 (KMS-encrypted)"]
                S3A[attachments bucket]
                S3B[backups bucket]
                S3AUD["audit bucket<br/>🔒 Object Lock 6 years"]
            end
            SSM["🔑 SSM Parameter Store<br/>(secure secrets)"]
            KMS["🗝️ KMS Customer Managed Keys<br/>(auto-rotation enabled)"]
            CW["📊 CloudWatch Logs<br/>(centralized, encrypted)"]
            IAM["👤 IAM Role<br/>(least privilege)"]
        end

        subgraph COMPLIANCE["🛡️ Compliance & Audit"]
            BAA["📜 AWS BAA Signed<br/>HIPAA-enabled"]
            CT["🔍 CloudTrail<br/>(multi-region audit)"]
        end
    end

    subgraph LEGACY["💀 Legacy Transpedite (current system)"]
        LEG["Continues running on current server<br/>Will be replaced in Sprint 7"]
    end

    %% Connections
    USER -->|HTTPS| INTERNET
    INTERNET -->|HTTPS 443| R53
    R53 --> EIP
    EIP --> NIPIO
    NIPIO -->|HTTPS| SG
    SG --> CADDY

    IGW <--> PUBSUB
    NAT --> PRIVSUB

    EC2 -.->|hosts| DOCKER
    EC2 -.->|assumes| IAM
    IAM -.->|read| SSM
    IAM -.->|use| KMS
    IAM -.->|rw| S3
    IAM -.->|put| CW

    RAILS -.->|encrypted| SSM
    POSTGRES -.->|backups| S3B
    RAILS -.->|attachments| S3A
    RAILS -.->|audit logs| S3AUD

    LEGACY -.->|🔄 Sprint 7 cutover| EC2

    classDef encryption fill:#e1f5ff,stroke:#0277bd,stroke-width:2px;
    classDef compliance fill:#fff3e0,stroke:#e65100,stroke-width:2px;
    classDef legacy fill:#f5f5f5,stroke:#9e9e9e,stroke-dasharray:5 5,color:#666;
    classDef future fill:#f3e5f5,stroke:#7b1fa2,stroke-dasharray:3 3;

    class S3AUD,POSTGRES,RAILS,SSM,KMS encryption;
    class BAA,CT compliance;
    class LEG,LEGACY legacy;
    class FUT,PRIVSUB future;
```

---

## Architecture in plain language

### 🌍 External
End users (Med-Rok hospital admins, case managers, intake persons) access the system through the public Internet over HTTPS.

### 🛡️ AWS Edge
- **Route 53**: AWS-managed DNS. Will hold the client's real domain when provided.
- **Elastic IP**: a stable public IP attached to the EC2 instance.
- **nip.io**: a free public DNS service used during Sprint 1. It resolves IP-based hostnames automatically — no domain registration needed for the demo phase.

### 🔵 VPC + Subnets
A dedicated AWS Virtual Private Cloud isolates Transpedite from any other systems in the AWS account:
- **2 Availability Zones** for future high-availability
- **Public subnets** host the application EC2 and the NAT Gateway
- **Private subnets** are reserved for future database services (RDS) and container orchestration (ECS) starting Sprint 2
- **Security Group**: only TCP 80/443 from the internet is allowed. **No SSH port is open** — operational access is via AWS SSM Session Manager.

### 💻 EC2 + 🐳 Docker stack

A single EC2 instance (Ubuntu 22.04 LTS, EBS-encrypted disk) runs the entire application stack via Docker Compose:

| Service | Purpose |
|---|---|
| **Caddy 2** | HTTPS termination, automatic Let's Encrypt certificates, security headers (HSTS, X-Frame-Options, etc.), reverse proxy to Rails |
| **Rails 7.2** | Application server (Puma 6, Ruby 3.3) |
| **PostgreSQL 16** | Database with TLS, scram-sha-256 authentication, 3 separate databases (development/test/production), 3 users with differentiated permissions |
| **Redis 7** | Cache and background-job queue (Sidekiq), password authentication, on-disk persistence |

### ⚙️ AWS Supporting Services

| Service | Purpose |
|---|---|
| **S3** (3 buckets, KMS-encrypted) | Patient attachments, Postgres backups, audit logs (audit bucket has Object Lock 6 years per HIPAA) |
| **SSM Parameter Store** | All secrets stored as SecureString with KMS encryption (DB passwords, Rails secrets, encryption keys) |
| **KMS** | 5 Customer Managed Keys with automatic annual rotation, one per use case |
| **CloudWatch Logs** | Centralized application logging, encrypted with KMS, retention policies enforced |
| **IAM** | The EC2 assumes a least-privilege role granting access only to its own resources |

### 🛡️ Compliance & Audit
- **AWS BAA signed** at the Organization level → all of AWS is covered by the HIPAA contract.
- **CloudTrail**: multi-region audit of all AWS API activity. Required by HIPAA Security Rule §164.312(b).

### 💀 Legacy (current system)
The current Transpedite system continues running on the client's existing infrastructure during the entire modernization project. The new v2 system is built **in parallel** — never modifying the current system in-place. Migration of data is the final step in **Sprint 7**, executed as a snapshot + delta + DNS switchover + 30-day legacy read-only fallback period.

---

## Encryption layers (defense in depth)

```mermaid
flowchart LR
    subgraph LAYER4["🔐 Layer 4: Field-level (Active Record Encryption)"]
        F1["Patient name, DOB,<br/>SSN digits, diagnosis,<br/>and other PHI fields"]
    end
    subgraph LAYER3["🔐 Layer 3: TLS Rails ↔ Postgres"]
        L3["sslmode=require<br/>TLS 1.2+"]
    end
    subgraph LAYER2["🔐 Layer 2: Postgres SSL"]
        L2["ssl = on<br/>TLS 1.2+"]
    end
    subgraph LAYER1["🔐 Layer 1: EBS encryption"]
        L1["AES-256 with KMS CMK<br/>at-rest disk encryption"]
    end

    LAYER4 --> LAYER3
    LAYER3 --> LAYER2
    LAYER2 --> LAYER1

    style LAYER4 fill:#ffebee,stroke:#c62828
    style LAYER3 fill:#fff3e0,stroke:#e65100
    style LAYER2 fill:#fffde7,stroke:#f9a825
    style LAYER1 fill:#e8f5e9,stroke:#2e7d32
```

If any single layer fails, the others continue protecting Protected Health Information (PHI). This satisfies HIPAA Security Rule §164.312 (Technical Safeguards — Encryption and Decryption).

---

## Request flow (what happens on each request)

```mermaid
sequenceDiagram
    actor User as 👨‍⚕️ User
    participant DNS as 🌐 DNS
    participant Caddy as 🔐 Caddy
    participant Rails as 💎 Rails 7.2
    participant PG as 🐘 Postgres 16
    participant Redis as 🔴 Redis 7
    participant SSM as 🔑 SSM
    participant S3 as 🪣 S3
    participant KMS as 🗝️ KMS

    User->>DNS: GET https://transpedite.example
    DNS->>User: Public IP address
    User->>Caddy: HTTPS request (TLS 1.2+)
    Note over Caddy: Let's Encrypt cert<br/>auto-renewed
    Caddy->>Rails: HTTP localhost:3000 (internal)
    Rails->>PG: SELECT * FROM cases (TLS)
    PG->>KMS: Decrypt encrypted columns
    KMS-->>PG: Plain values
    PG-->>Rails: Rows (with PHI decrypted)
    Rails->>Redis: Cache lookup (with auth)
    Redis-->>Rails: Cached response
    Rails->>S3: GET /attachments/file (presigned URL)
    S3->>KMS: Decrypt object
    KMS-->>S3: Plain bytes
    S3-->>Rails: File content
    Rails-->>Caddy: HTML response
    Caddy-->>User: HTTPS response<br/>+ HSTS, X-Content-Type, etc.

    Note over Rails: All access logged to<br/>append-only audit table
```

---

## Project timeline

```mermaid
gantt
    title Transpedite v2 — Sprint Timeline
    dateFormat YYYY-MM-DD
    axisFormat %b %d

    section Sprint 1 (current)
    AWS Infrastructure        :done, infra, 2026-05-11, 2d
    Docker stack deployed     :done, stack, 2026-05-12, 1d
    HTTPS + Let's Encrypt     :done, tls, 2026-05-12, 1d
    Client demo               :crit, demo, 2026-05-13, 1d
    PHI Gate documented       :active, phi, 2026-05-14, 2d
    Migration plan            :active, mig, 2026-05-14, 2d

    section Sprint 2 (Foundation)
    Auth + MFA + RBAC         :s2auth, 2026-05-18, 5d
    Backups automated         :s2bk, 2026-05-18, 5d
    Audit logging             :s2au, 2026-05-21, 5d

    section Sprint 3-4 (Domain Core)
    Models: Case, Bed, Match  :s3, 2026-06-01, 14d
    Statesman state machines  :s3sm, 2026-06-01, 14d

    section Sprint 5 (Workflows)
    Matching engine           :s5m, 2026-06-15, 14d
    Notifications             :s5n, 2026-06-15, 14d
    Search                    :s5s, 2026-06-22, 7d

    section Sprint 6 (Client modules)
    Discharge Barrier         :s6db, 2026-06-29, 7d
    Inpatient Psych           :s6ip, 2026-06-29, 7d

    section Sprint 7 (CUTOVER)
    Migration scripts         :s7m, 2026-07-13, 7d
    PHI Gate (14 controls)    :crit, s7pg, 2026-07-20, 4d
    Penetration test          :crit, s7pen, 2026-07-20, 4d
    Cutover + Hypercare       :crit, s7co, 2026-07-27, 14d
```

---

## What's deployed today (Sprint 1)

| Aspect | Status |
|---|---|
| AWS infrastructure provisioned via Infrastructure-as-Code (Terraform) | ✅ Done |
| Docker stack (Caddy + Rails + Postgres + Redis) running on EC2 | ✅ Done |
| HTTPS with auto-renewed Let's Encrypt certificate | ✅ Done |
| AWS BAA signed at Organization level | ✅ Done |
| Encryption at-rest, in-transit, and field-level (4 layers) | ✅ Configured |
| PHI Gate of 14 mandatory HIPAA controls | 🟡 Documented, in progress |
| Migration plan from legacy | 🟡 In progress |
| Real PHI loaded into system | ❌ Not yet (gated by PHI Gate completion in Sprint 7) |

---

> **Note**: This diagram is updated at the end of each sprint to reflect the current architecture. The next snapshot will be added after Sprint 2.
