# Systems Manager Patch Manager
This tool automates patching of your EC2 and on-premises servers.
**What does it handle?**
 ✅ Automatically scans for missing patches
 ✅ Defines patch rules (baselines)
 ✅ Groups servers for patching (Patch Groups)
 ✅ Applies patches in a controlled, environment-aware way
 Key Concepts *Systems Manager Patch Manager*
 1️⃣ *Patch Baseline*
 * This defines which patches should be applied.
Example:
Baseline for Dev: all non-critical + critical patches
Baseline for Prod: only critical patches, with a 7-day wait
💡 Think of it as your “approved patch rulebook” per environment.
2️⃣ Patch Group
* Logical grouping of EC2 instances based on tags.
* **MUST use the tag key: Patch Group** (this is an AWS rule).
Example:
    Dev instances → Patch Group = Dev-Patch
    Prod instances → Patch Group = Prod-Patch
💡 Patch Groups connect specific servers to specific baselines.
3️⃣ Tagging
* You must tag EC2 instances properly:
 ✅ By environment
 ✅ By OS type (if needed)
 Example:
Environment: Dev, Patch Group: Dev-Patch
Environment: Prod, Patch Group: Prod-Patch
💡 Without the right tags, Patch Manager can’t target the right servers.
4️⃣ Patch Compliance
After patching, you can verify what was patched, what failed, and what’s missing.

# VPC & Networking (Peering, TGW, Direct Connect)
1️⃣ Trap: VPC Peering is NOT transitive.
 ➡️ Even if A ↔️ B and B ↔️ C, A 🚫 cannot talk to C.
2️⃣ Trap: Direct Connect does NOT allow VPC-to-VPC traffic.
 ➡️ It connects on-premises to AWS—NOT VPC-to-VPC. Use Transit Gateway + Direct Connect Gateway for multi-VPC routing.
#  IAM & Security (Cross-Account Access, Policies)
1️⃣ Trap: Resource-based policies allow cross-account access WITHOUT switching roles.
    ➡️ Identity-based policies need AssumeRole.
2️⃣ Trap: Trust Policy’s Principal must point to the external account/user ARN—NOT to your own account or role name.

# CloudFront & Encryption
1️⃣ Trap: Field-Level Encryption is needed when encrypting specific sensitive fields (e.g., credit card numbers), not just HTTPS.
2️⃣ Trap: Adding headers (User-Agent, Host) to CloudFront cache key can reduce cache hit ratio, NOT improve it.

# Migration Strategies (6 R’s)
1️⃣ Trap: If you're migrating NOW with no code changes, it’s always Rehost—even if you plan to refactor later
2️⃣ Trap: Replatform = minor tweaks now (e.g., moving DB to RDS)—NOT a full rewrite (that's Refactor).

🧠 **Blue/Green works** best when you can duplicate everything and switch over cleanly.
❌ It’s not ideal when apps need live state coordination, feature flags, or have tight upgrade paths (like COTS apps).

# Security Groups vs NACLs
1️⃣ Trap: Security Groups are stateful (return traffic auto-allowed); NACLs are stateless (must allow return traffic explicitly).
2️⃣ Trap: NACLs evaluate rules in order (lowest # first). First match WINS. SGs evaluate all rules together.

# Direct Connect & VPN
1️⃣ Trap: VPN is over the internet (IPSec encrypted), while Direct Connect is a dedicated private connection.
2️⃣ Trap: Public VIF (Virtual Interface) on Direct Connect is for public AWS services (e.g., S3), NOT for private VPC access. For VPC access, you need a Private VIF.

# S3 Access & Encryption
1️⃣ Trap: Bucket policies (resource-based) can grant cross-account access directly. IAM user policies alone can’t do this unless combined with AssumeRole.
2️⃣ Trap: S3 encryption:
SSE-S3 = AWS manages keys
SSE-KMS = You manage keys via KMS
 ➡️ **SSE-KMS requires extra permission to use the key (kms:Decrypt, etc.).**
 # IPSEC VPN and VPC 

 1️⃣ IPSec tunnels are built for “on-prem to AWS,” not AWS VPC to AWS VPC.
 2️⃣ Instead of IPSec, AWS gives you:
  * VPC Peering
  * Transit Gateway (TGW)

➡️ These are AWS-native solutions that use the AWS backbone network—they don’t need IPSec tunnels.

| Scenario                                              | Solution(s)                                  | Notes                                  |
|------------------------------------------------------|----------------------------------------------|----------------------------------------|
| Connect on-prem <--> AWS VPC securely (encrypted)    | ✅ Site-to-Site VPN (IPSec tunnel)           | AWS Native Solution                    |
| Connect VPC <--> VPC (same region or inter-region)   | ✅ VPC Peering OR Transit Gateway            | NO IPSec tunnel involved               |
| Connect multiple VPCs + scale easily                 | ✅ Transit Gateway (TGW)                     |                                        |
| Connect multi-region + on-prem in one go             | ✅ Direct Connect + Direct Connect Gateway   |                                        |

🚨 KEY PHRASE TO MEMORIZE:
 👉 “AWS does NOT provide native IPSec VPN tunnels between two VPCs; use VPC Peering or Transit Gateway instead.”


 | AWS Config                                                                 | AWS Systems Manager (SSM)                                           |
|----------------------------------------------------------------------------|---------------------------------------------------------------------|
| 🛡️ **Think:** Continuous Compliance & Auditing                             | 🔧 **Think:** Managing, Operating & Patching your infrastructure    |
| Tracks & records the state of your resources (configuration history).       | Lets you manage servers, install patches, run commands, automate ops tasks. |
| Continuously checks if resources meet compliance rules.                     | Example: “Patch my EC2 Linux machines. Run a script across all servers.” |
| Example: “Are my S3 buckets public? Is my IAM policy too open?”            |                                                                     |
| You ask: “Is my AWS resource configured correctly & compliant?”             | You ask: “How do I manage and operate my servers and services?”     |
| ✅ **Best for:** Auditing, compliance, security monitoring.                 | ✅ **Best for:** Automation, maintenance, operational tasks.         |

🚨 The BIG TRAP to Remember for the Exam:
👉 **If the question is about:**
❓ Compliance, auditing, continuously checking policies or configs:
 ➡️ ✅ **AWS Config is the answer.**


❓ Running commands, patching servers, automating operational tasks:
 ➡️ ✅ **WS Systems Manager (SSM) is the answer.**


💡 Easy phrase:
🔑 “Config = Compliance. SSM = SysAdmin tasks.”
✅ When question says:
“User keeps working in their account + no need to switch roles” ➔ ✅ Resource-based policy.

# 🚀  How to Improve Cache Hit Ratio:

| Technique                                 | Why It Helps                                                                                   |
|--------------------------------------------|-----------------------------------------------------------------------------------------------|
| Cache-Control max-age (longer TTL)         | The longer CloudFront can cache content, the fewer times it goes back to the origin.           |
| Minimize unnecessary headers/query strings | CloudFront caches per unique request. Extra headers/query strings can cause cache misses.      |
| Compress objects (gzip/brotli)             | Reduces size = faster = more cache-friendly.                                                   |
| Use Origin Shield                          | Adds another caching layer closer to your origin, improving cache hit ratio for multi-region requests. |
| Versioned URLs (e.g., /v1/logo.png)        | Lets you cache forever (long TTL) and force refresh only when you change the file (by changing the URL). |

# AWS Security services usecase

| Use Case                                      | AWS Service            | Included/Status     |
|-----------------------------------------------|------------------------|---------------------|
| Infra-layer DDoS protection (SYN, UDP)        | AWS Shield Advanced    | ✅                  |
| Track and enforce resource config compliance  | AWS Config             | ✅                  |
| App-level threats (SQLi, XSS, bots)           | AWS WAF                |                     |
| Org-wide firewall/DDoS policy enforcement     | AWS Firewall Manager   |                     |
| Patch, automate EC2 ops                       | AWS Systems Manager    |                     |

🧱 WAF Core Building Blocks

| Component                    | What It Does                                                                                           |
|------------------------------|--------------------------------------------------------------------------------------------------------|
| Web ACL (Access Control List) | The main WAF rule group you attach to resources (like ALB, API Gateway, CloudFront)                    |
| Rules                        | Logic like "block all SQLi", "allow only US IPs", "rate limit per IP"                                  |
| Rule Groups                  | Reusable sets of rules, like AWS Managed Rule Groups (pre-built protections)                           |


| Scenario                          | Federation Type           | STS Call                  |
|------------------------------------|---------------------------|---------------------------|
| Enterprise/On-Prem Users (AD, ADFS)| ✅ SAML Federation         | AssumeRoleWithSAML        |
| Mobile/Web App Users (Google, FB)  | ❌ Web Identity Federation | AssumeRoleWithWebIdentity |
| OAuth 2.0 Access                   | ❌ Not supported for console login | —                |

“SAML for corporate users, Web Identity for app users.”
🔑 “STS + AssumeRoleWithSAML = enterprise login via SSO”
Forward Proxy = the ONLY option that gives URL-level outbound control from EC2 instances

🧠 Trigger Phrase for VPN CloudHub in Exam:
“multiple remote sites” + “VPN” + “hub-and-spoke” + “cost-effective”
"Want to block direct S3 access and force CloudFront access? → Use OAI

 security groups and network ACLs. Security groups control inbound and outbound traffic for your instances, and network ACLs control inbound and outbound traffic for your subnets

AWS Storage Gateway iSCSI security against replay or spoofing attacks?
✅ Enable CHAP authentication for initiator and target

# EBS

| Data Pattern                      | Volume Type         |
|------------------------------------|--------------------|
| Random access (DB)                 | gp3/gp2 or io1     |
| Sequential access (logs, streaming)| st1                |
| Infrequent, archived, cold         | sc1                |

# Migration tool

| Use Case                | Migration Tool | Storage Type                                         |
|-------------------------|---------------|------------------------------------------------------|
| Full server (VMs, whole OS) | SMS           | Based on general disk type                           |
| Database only           | DMS           | Match volume type to access pattern (gp2, io1, st1, sc1) |


## When to Use Signed Cookies vs Signed URLs

| Use Case                                 | Use Signed URLs | Use Signed Cookies |
|------------------------------------------|:--------------:|:-----------------:|
| User needs access to one file            |      ✅ Yes     |      ❌ Not ideal  |
| User needs access to many files          |      ❌ No      |      ✅ Yes        |
| You don’t want to change URLs            |      ❌ No      |      ✅ Yes        |
| Protecting video libraries, images, docs |      ❌ No      |      ✅ Yes        |


---

## Mobile App Authentication Best Practices

- Mobile apps that use Google/Facebook login = **Web Identity Federation + STS + temporary creds**
- ❌ **Never** hardcode access keys into mobile apps.
- ✅ **Always** let users log in with their identity provider and get temporary access to AWS.


| Federation Type         | Use Case                                                                 | STS API Used                      | Credential Type                           | Best For                                           | Security Risk                           |
|-------------------------|--------------------------------------------------------------------------|-----------------------------------|-------------------------------------------|----------------------------------------------------|------------------------------------------|
| Web Identity Federation | Mobile/web apps using Google, Facebook, or OIDC login                   | AssumeRoleWithWebIdentity         | Temporary credentials (via STS)           | Apps where users log in with social accounts       | Low (uses short-lived tokens)           |
| SAML Federation         | Enterprise users with corporate credentials (e.g., ADFS, AD)            | AssumeRoleWithSAML                | Temporary credentials (via STS)           | Enterprise SSO and centralized user management     | Low (tokens expire)                     |
| Cognito                 | User sign-up/sign-in with AWS-hosted identity/user pools                | Cognito handles internally with STS | Temporary credentials (via STS)           | Consumer apps with user directory and auth mgmt    | Low (tokens + user pool auth)           |
| IAM Users               | Static, long-lived AWS identities (not federated)                       | Not applicable                    | Long-term access keys (manually rotated)  | Scripts, CLI use, admin users (bad for apps)       | High (long-term keys can be leaked)     |

# Ldap and Identity broker

✅ What is an Identity Broker?
An identity broker is like a translator and gatekeeper between:
🧑 Your corporate identity system (LDAP, AD, etc.)

🔄 It does 3 main jobs:
Authenticate the user using LDAP


Map the authenticated user to an IAM Role in AWS


Request temporary AWS credentials from STS using AssumeRole


☁️ AWS Security Token Service (STS)

Your identity broker sits between LDAP and AWS


It verifies the user credentials against LDAP


If successful, it calls AWS STS (AssumeRole) and gives back temporary AWS credentials


The web app calls the broker and uses the credentials returned to access AWS

**User → Web App → Identity Broker → LDAP + STS → Temporary AWS creds**
**Mental model: LDAP → Broker → STS → AWS creds**

| Step | What Happens                                               |
|------|-----------------------------------------------------------|
| 1️⃣   | User logs into web app with LDAP credentials              |
| 2️⃣   | Web app or broker verifies credentials against LDAP       |
| 3️⃣   | If valid, it calls AWS STS: AssumeRole                   |
| 4️⃣   | Gets temporary credentials for that IAM role              |
| 5️⃣   | App uses those creds to access S3, DynamoDB, etc.         |

Memory Hook
**“LDAP can’t talk to IAM directly—STS is the translator, and the identity broker is the bouncer.”**

| Component      | Purpose                                  |
|----------------|------------------------------------------|
| LDAP           | Corporate user authentication            |
| Identity Broker| Maps LDAP user to IAM role and calls STS |
| STS            | Issues temporary AWS credentials         |
| IAM Role       | Controls what the user can access in AWS |

✅ Benefits of Identity Broker
🔐 No need for long-term IAM users or access keys
🔁 Temporary credentials that auto-expire
🛡️ Strong access control using IAM roles
🔄 Works with any identity provider (LDAP, SAML, OIDC)

| Concept                | Example                                                        |
|------------------------|----------------------------------------------------------------|
| You control the key    | You create it yourself, AWS never stores it                    |
| You send the key       | Every time you upload/download, you must include the key       |
| AWS encrypts for you   | But only temporarily, then forgets the key                     |
| Headers matter         | You must send 3 headers with API/SDK/CLI calls                 |


"You can’t change your VPC’s original block, but you CAN expand it with 4 more CIDRs."
 CIDR expansion ≠ VPC peering.

 # RDS 

 **Why CNAME, Not IP Address?**
RDS gives your DB instance a CNAME DNS endpoint like:

 CopyEdit
mydb.abcdefghij.us-east-1.rds.amazonaws.com


This CNAME always points to the current primary


AWS flips it behind the scenes during failover

That’s why apps should **always connect using the RDS endpoint** and not the underlying IP—because IPs change on failover, but the **CNAME stays the same.*

Multi-AZ = standby is on standby. If primary dies, AWS promotes standby and flips the CNAME—fast, smooth, automatic.

# 🧠 RDS High Availability & Scaling Cheat Sheet

| Feature               | Multi-AZ Deployment                              | Read Replica                                  | Global Database                              |
|----------------------|--------------------------------------------------|-----------------------------------------------|----------------------------------------------|
| 🔄 Purpose            | High availability (failover support)             | Read scalability                              | Cross-region disaster recovery & reads       |
| 👨‍💻 Usage Pattern     | Production databases needing HA                  | Reporting, analytics, read-heavy workloads     | Global apps with low-latency read access     |
| 🔃 Replication Type   | Synchronous (real-time replication)              | Asynchronous                                  | Asynchronous                                  |
| 📚 Used For Reads     | ❌ No (standby cannot serve traffic)              | ✅ Yes (you can connect to read replicas)       | ✅ Yes (cross-region reads supported)         |
| 🔧 Writes Allowed     | ✅ Primary only                                   | ❌ No (read-only copies)                       | ❌ No (except primary region)                 |
| ⛑️ Failover Support   | ✅ Automatic failover to standby                  | ❌ Not automatic (must manually promote)       | ✅ Manual or semi-automatic regional failover |
| 🌐 Region Support     | Single region                                    | Single region (or cross-region)               | ✅ Multi-region                               |
| 📎 Endpoint Type      | One endpoint (CNAME, auto-flips on failover)     | One per replica                               | One per region                               |
| ⚠️ Common Mistakes    | Confused with read scaling                       | Not HA – no automatic failover                | Not suitable for fast write replication      |
| 💸 Cost Implication   | Charged for both primary & standby               | Charged per replica                           | Higher cost due to replication & latency     |
| 🧪 Best Use Case      | Tier-1 production apps needing 99.99% uptime     | Apps with lots of read traffic                | Multi-region, global-critical workloads       |

**✅ Key Takeaways:**

- **Multi-AZ = HA**, **not** read scaling.
- **Read Replica = Scaling**, **not** HA.
- **Global DB = Worldwide scale**, **disaster resilience**.

> ⚡ Always use the **RDS endpoint (CNAME)** when connecting. AWS auto-flips it during failover in Multi-AZ.

## Key FSx Variants and Their Use Cases

| FSx Variant               | Description                                                                                   | Ideal Use Cases                                                      |
|---------------------------|----------------------------------------------------------------------------------------------|---------------------------------------------------------------------|
| FSx for Windows File Server | Fully managed Windows file system with support for SMB protocol and Active Directory integration. | Windows-based applications, home directories, lift-and-shift migrations. |
| FSx for Lustre            | High-performance file system optimized for fast processing of workloads.                     | Machine learning, high-performance computing (HPC), and big data analytics. |
| FSx for NetApp ONTAP      | Provides NetApp's ONTAP file system capabilities with multi-protocol access.                 | Enterprises needing NetApp features like SnapMirror and FlexClone.   |
| FSx for OpenZFS           | Offers OpenZFS file system with features like snapshots and data cloning.                    | Applications requiring ZFS features, data protection, and efficient data cloning. |

---

## M-A-C-I-E = “Monitoring And Classification In Everything”

**Amazon Macie:**
- Detects PII
- Flags risky data storage
- Classifies data in S3 buckets

---

## CloudFront + HTTPS (Viewer Layer)

✅ **Use Viewer Protocol Policy:**
- Redirect HTTP to HTTPS (best UX)
- HTTPS Only (most secure)

✅ **Use CloudFront’s default SSL cert** (for `*.cloudfront.net`)

❌ **Do NOT use self-signed certs** at ELB or S3 with CloudFront

❌ **ELB doesn’t have a default cert**; needs ACM or imported cert

> Use CloudFront + Viewer Protocol Policy to enforce HTTPS and boost SEO 🔐

---

## GuardDuty and Trusted IPs

- GuardDuty only trusts IPs, not EC2 instance IDs.
- To suppress findings from specific EC2s, give them Elastic IPs and add those IPs to the Trusted IP list.
