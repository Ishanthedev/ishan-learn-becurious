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
 ➡️ It connects on-premises to AWS—NOT VPC-to-VPC. Use Transit Gateway + 

 Direct Connect Gateway for multi-VPC routing.
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


## Section Migration and Modernization


# Correct DNS Rule Summary for Route 53

**MEMORIZE THIS TABLE**

| AWS Resource Type           | Route 53 Record Type      | Alias Allowed? | Notes                            |
|----------------------------|---------------------------|:-------------:|----------------------------------|
| EC2 instance               | A record (non-alias)      |      ❌        | Use IP address directly          |
| Elastic Load Balancer (ELB)| A record (Alias)          |      ✅        | Must use Alias A record          |
| CloudFront                 | A record (Alias)          |      ✅        | Required for apex domains        |
| S3 Static Website          | A record (Alias)          |      ✅        | Needs static hosting enabled     |
| RDS Endpoint               | CNAME only                |      ❌        | AWS manages RDS CNAME            |

Route 53 + ELB = Alias A Record (Not regular A Record)
When routing to ELB, use Alias A record so AWS handles DNS resolution dynamically.
example.com → Alias A → ELB DNS (xyz.us-east-1.elb.amazonaws.com)


This allows DNS to automatically follow fail overs and changes.

# Application Migration Service (MGN) vs Database Migration Service (DMS)

| Service | Purpose                                      |
|---------|----------------------------------------------|
| **MGN** | Lift-and-shift VMs/servers (like EC2 migrations) |
| **DMS** | Migrate databases (like MySQL to RDS)           

**Memorization Trick for Route 53:**
```
"ELB, S3, CloudFront – Alias A is the Front.
 RDS – Use CNAME, No Alias zone."
 ```

 MGN → for mission-critical VMs
Keeps them online, automates cutover, minimal downtime

✔ Snowball → for bulk migration of non-critical VMs
40TB over 12Mbps = >310 days (would miss the 3-month deadline)


Snowball completes the data move in ~1 week

✔ VM Import/Export → to make them run as EC2
Converts your VMs to AMI format, boots them in AWS

# AWS Migration Decision Table

Use this table to quickly determine the recommended AWS migration tool or service based on your scenario:

| **Condition**                               | **Use**           |
|---------------------------------------------|-------------------|
| Limited internet + large data (TBs)         | Snowball          |
| Live VMs / low downtime                     | MGN               |
| Simple lift-and-shift + pre-exported VMs    | VM Import/Export  |
| Replatform/refactor                         | Only if explicitly asked |
| Large budget + need low latency             | Direct Connect    |

## Migration Strategy Cheat Sheet for AWS Pro

### Use AWS MGN when:
- VMs need low downtime
- Continuous replication required
- Lift-and-shift approach

### Use Snowball when:
- TBs or PBs of data
- Network is a bottleneck
- Cost-effective offline transfer needed

### Use VM Import/Export when:
- VMs already exported
- No real-time cutover needed

### Avoid Direct Connect when:
- Budget is tight
- No ongoing hybrid needs

# CORE CONCEPT: When to Use AWS Application Migration Service (MGN)

| Requirement                        | Why MGN Fits                                                                                 |
|-------------------------------------|---------------------------------------------------------------------------------------------|
| ✅ Import on-prem VMs               | MGN supports lift-and-shift from VMware, Hyper-V, and physical servers                      |
| ✅ Sync changes until cutover       | MGN uses continuous block-level replication via lightweight agents                          |
| ✅ Terabytes of root + data volume  | It replicates all attached volumes automatically                                            |
| ✅ Minimal downtime                 | After testing, you cutover with near-zero downtime                                          |
| ✅ Minimal ops overhead             | Fully automated, no need for custom scripts like in VM Import/Export                        |

# REMEMBER FOR EXAM: MGN vs VM Import/Export

| Feature                | MGN (AWS Application Migration Service) | VM Import/Export         |
|------------------------|:--------------------------------------:|:------------------------:|
| Continuous sync        | ✅ Yes                                 | ❌ No                    |
| Minimal downtime       | ✅ Yes                                 | ❌ High downtime (re-import entire image) |
| Multi-volume support   | ✅ All attached volumes                | ❌ Manual snapshots needed|
| Automation             | ✅ Highly automated                    | ❌ Manual scripts or CLI  |
| Best for               | Lift-and-shift, rehosting              | One-time imports         |
| Replication agent needed? | ✅ Yes                              | ❌ No                    |

# AWS Application Migration Service (MGN)

## Use Case:
- Lift-and-shift migration of on-prem VMs (VMware, Hyper-V, physical)
- Minimal downtime required
- Sync changes until production cutover

## Benefits:
- Block-level continuous replication
- Automates launch of EC2 from replicated images
- Supports data + root volumes
- Test + Cutover workflows built-in
- Zero manual re-imports or scripting needed

## Key Differences:
- MGN > VM Import/Export when downtime is unacceptable
- MGN supports re-platform/refactor later

## Important:
- Install replication agent
- Plan launch templates per instance
- Cutover stops replication and finalizes migration

**🛠 Key Workflow with MGN**
Install agent on source servers (Windows/Linux)
MGN does block-level replication
Perform Test launch from AWS
Validate functionality
Do a Cutover launch
Decommission old systems 🎯

# AWS Storage Gateway – Tape Gateway

## Purpose:
Modernize on-premises **tape backups** by moving to **Amazon S3 + Glacier** using virtual tapes.

## How It Works:
1. Existing backup software (e.g., Veeam, NetBackup) writes to **virtual tape** (iSCSI)
2. Tape data is stored in **Amazon S3**
3. Archived tapes are moved to **Virtual Tape Shelf (VTS)** in **Amazon Glacier**

## Benefits:
- Minimal changes to legacy backup processes
- High durability (99.999999999%) via Glacier
- Pay-as-you-go archival storage
- Avoids physical tape management

## Notes:
- **VTS** = Glacier storage location for virtual tapes
- Can't retrieve archived tapes immediately – must restore first
- **Don't use File Gateway or Volume Gateway** for tape backups
Use AWS Schema Conversion Tool (SCT) + AWS Database Migration Service (DMS)

This is the official AWS 2-step solution for heterogeneous database migration (Oracle → PostgreSQL, SQL Server → Aurora, etc.).
Example Migration Flow (Oracle → PostgreSQL):

CopyEdit
1. Use **SCT** to convert schema (tables, indexes, stored procs) → PostgreSQL-compatible
2. Deploy the converted schema to target RDS PostgreSQL
3. Use **DMS** to:
   - Do full load (migrate data)
   - Optionally enable CDC (change data capture) for near-zero downtime
4. Cutover when app is ready

**What is AWS DataSync?**
🔧 Purpose:
High-speed, automated data transfer service to move files to/from AWS.
✅ Use Cases:
Migrate SMB/NFS shares (e.g., your on-prem file servers)


Move logs, backups, archives to S3 / EFS / FSx


Daily data sync between on-prem and cloud

## Key Features

| Feature                | Description                                                                                  |
|------------------------|----------------------------------------------------------------------------------------------|
| ✅ SMB/NFS support     | Can directly connect to your on-prem Windows/Linux file shares                               |
| 🔄 Incremental Syncs   | Only transfers changed data after the first run                                              |
| 🚀 High performance    | Uses purpose-built agent for 10x faster transfer than scripts                                |
| 🔐 Secure + scheduled  | Uses TLS, IAM roles, and supports automated scheduling                                       |

Where it Sends Data:
S3 buckets
Amazon FSx (Windows File Server)
Amazon EFS

**💡 Part 2: What is VM Import/Export?**
🔧 Purpose:
Convert on-prem VM images (e.g., VMware, Hyper-V) into EC2 instances.
✅ Use Cases:
Lift-and-shift VM-based apps
Preserve full software/config state
Create custom EC2 AMIs from VMs
**🧬 Supported formats:**
OVA
VMDK
VHD
**📝 Requirements:**
Upload VM image to S3
Create IAM role with import permissions
Use AWS CLI or API to import and convert into EC2 AMI

# AWS DataSync + VM Import/Export

## Use Case:
Migrate SMB shares + VMware VMs to AWS

## DataSync:
- For SMB/NFS file transfer
- Transfers data to S3, FSx, or EFS
- Scheduled, incremental, secure
- Uses on-prem **DataSync Agent**

## VM Import/Export:
- Upload .vmdk/.vhd/.ova to S3
- Converts to EC2 AMI
- Use AWS CLI to import and launch EC2

## Exam Tip:
✅ Use **DataSync** for file server migration  
✅ Use **VM Import/Export** for full VM lift-and-shift

# 🧠 Amazon SQS vs. Amazon MQ – AWS SA Pro Exam Comparison

| Feature                         | Amazon SQS                                                | Amazon MQ                                                  |
|---------------------------------|------------------------------------------------------------|------------------------------------------------------------|
| **Purpose**                     | Fully managed message queue service                        | Managed message broker service                             |
| **Best For**                    | New apps that need scalable, decoupled messaging           | Legacy apps requiring compatibility with existing brokers  |
| **Protocols Supported**         | Proprietary AWS API (send/receive/delete messages)         | Industry-standard protocols: AMQP, MQTT, OpenWire, STOMP   |
| **Queue Types**                 | Standard (at-least-once), FIFO (exactly-once)              | Topics and queues (JMS-like model)                         |
| **Scalability**                 | Virtually unlimited, serverless                            | Scales vertically; not suitable for high TPS (by default)  |
| **Ordering Guarantees**        | FIFO queue for strict ordering                             | Supports ordering via topics or queues                     |
| **Latency**                     | Low latency (millisecond)                                  | Higher than SQS; depends on broker health                  |
| **Management Overhead**        | Minimal (no server to manage)                              | Medium (broker configuration, failover handling)           |
| **Durability**                  | Highly durable (SQS stores messages across AZs)            | Durable based on broker configuration                      |
| **Retry/Dead Letter Support**  | Built-in retry and DLQ support                             | Retry/DLQ managed by broker policy                         |
| **Monitoring**                  | CloudWatch metrics for queue length, age, etc.             | CloudWatch + detailed broker logs                          |
| **Message Retention**          | Up to 14 days                                              | Depends on broker settings                                 |
| **Cost Efficiency**            | Pay-per-request, very low-cost                             | Charged per broker instance and throughput                 |
| **Use Case Examples**          | Decoupled microservices, background jobs, event queues     | Migrating JMS-based apps to cloud, on-prem broker lift-n-shift |
| **Recommended When**           | You’re building cloud-native apps or want massive scale    | You have existing broker-dependent systems                 |
| **Not Ideal For**              | Apps needing MQTT/JMS/AMQP protocols                       | High-throughput modern cloud-native apps                   |

## 🔥 Exam Tip:
- Use **Amazon MQ** for **broker migration** or **JMS protocol compatibility**
- Use **Amazon SQS** for **serverless, decoupled, high-scale messaging**

**What is AWS Direct Connect?**

Direct Connect (DX) gives you a private, dedicated network connection between your on-prem data center and your AWS VPC.
Bypasses the internet
Low latency
Secure and reliable
Ideal for legacy workloads needing consistent throughput

🧪 Used when:
Applications require a private and dedicated connection

You need high bandwidth and low-latency for apps like SAP, Oracle, on-prem DBs, etc.

🔷 Why do I need BGP for Direct Connect?
BGP (Border Gateway Protocol) is the routing protocol used to exchange routes between:
Your on-prem router
AWS's Direct Connect router

Think of BGP as the GPS that lets both ends know:
"Hey, here’s how you can reach all these networks I own."
💡 BGP MD5 authentication is just a security layer ensuring the routers trust each other.
🔐 Without BGP → Your Direct Connect can’t route traffic.

On-Premises Data Center
        |
    [Router] -- BGP
        |
  AWS Direct Connect
        |
   Your VPC (Private IPs)
        |
   EC2 runs legacy app

### ❄️ AWS Snowball Edge – Performance Optimization

**Top Strategies to Speed Up Transfer:**
- 🔁 **Use multiple concurrent copy sessions** (most impact)
- 📦 **Batch small files** (< 1 MB) into `.zip`, `.tar`, `.tgz` archives
- 🚫 Avoid modifying/renaming files during transfer
- 🔗 Minimize network hops: source, Snowball, and transfer PC on same switch
- ⚡ Use high-speed NICs (10/40/100 Gbps), **not USB** (not supported)

**Remember:**
- Clustering = capacity & durability (not copy speed)
- File interface = file-level copy
- S3 Adapter = object-based programmatic upload (not faster for files)

## ❄️ AWS Snowball Edge – Transfer Performance Key Concepts

| Concept                         | Explanation                                                                 | Exam Hint                                                   |
|---------------------------------|-----------------------------------------------------------------------------|-------------------------------------------------------------|
| **Parallel Copy Sessions**      | Most impactful way to improve performance. Open multiple terminal sessions to write data in parallel. | ✅ Best choice when performance is slower than expected      |
| **Batch Small Files**           | Small files (under 1 MB) create high encryption overhead. Compress into `.tar`, `.zip`, or `.tgz` before copying. | ✅ Do this for millions of log files                         |
| **Use High-Speed Network Ports**| Use 10/40/100 Gbps Ethernet ports (not USB — Snowball Edge doesn’t support USB). | ❌ Never select "USB transfer" options                       |
| **Minimize Network Hops**       | Connect device + source + transfer terminal to the same switch. Avoid extra routers or switches. | 🔍 Improves real throughput, especially in busy networks     |
| **Keep Files Static During Transfer** | Do not rename, modify, or write to files during copy. It slows down transfer speed. | ⚠️ Avoid unnecessary filesystem changes mid-copy             |
| **Clustered Snowball Edge Devices** | Helps with storage capacity and durability, not speed of a single copy job. | ❌ Don’t assume clustering = faster ingestion   

How to NEVER Miss Snowball in Migration Scenarios Again
Here’s a simple mental checklist for spotting Snowball as the right solution:
💡 Snowball Trigger Rules (use if 1 or more is true):
>10TB data
Slow network connection (e.g., <100 Mbps)
Migration must happen quickly (≤ 1 month)
One-time or initial full data load
Files, DBs, or backups need physical transfer

💡 Snowball + DMS Hybrid Pattern (often tested):
Use Snowball for bulk initial data transfer
Use DMS for ongoing replication + near-zero-downtime cutover

“Big Data, Slow Pipe? Snowball Rides First. DMS Drives After.”
(Say this every time you see >10TB + VPN)
When you see:
“VMware” + “vCenter” + “want AMIs” or “move to EC2”

🔒 Always default to:
 ✅ AWS Application Migration Service (MGN)
 ✅ Install Replication Agent on VMs

 # IBM MQ vs Amazon MQ vs Amazon SQS

| Feature           | IBM MQ                          | Amazon MQ                                        | Amazon SQS                                    |
|-------------------|----------------------------------|--------------------------------------------------|-----------------------------------------------|
| **Purpose**        | Enterprise message broker        | Managed broker service compatible with IBM MQ    | Lightweight, scalable queue service           |
| **Protocols Supported** | JMS, MQTT, AMQP, STOMP          | Same as IBM MQ (JMS, STOMP, etc.)                | Proprietary only, no MQ compatibility         |
| **Migration Type** | Re-platform to Amazon MQ         | ✔️ 1:1 feature match, lift-and-shift ready         | ❌ Feature mismatch                            |
| **Use Case**       | Complex enterprise apps          | Legacy MQ migration to AWS                       | Modern microservices needing decoupled queues |
| **FIFO?**          | Yes                              | Yes                                              | Yes (Standard and FIFO queues available)      |

# AWS Cloud Migration Strategies – The 6 Rs

## 1. Rehost (“Lift and Shift”)
In a large legacy migration scenario where the organization is looking to quickly implement its migration and scale to meet a business case, the majority of applications are typically **rehosted**.

> 🔹 Fastest path to the cloud  
> 🔹 No changes to the application  
> 🔹 Ideal for large-scale migrations

## 2. Replatform (“Lift, Tinker and Shift”)
This entails making **a few cloud optimizations** in order to achieve tangible benefits without changing the **core architecture** of the application.

> 🔹 Example: Move database from self-managed to RDS  
> 🔹 Retains existing app structure  
> 🔹 Reduces operational burden

## 3. Repurchase (“Drop and Shop”)
This is a decision to **move to a different product**, often SaaS-based, and usually means your organization is **changing its licensing model**.

> 🔹 Example: Move CRM to Salesforce  
> 🔹 Good for eliminating legacy license overhead  
> 🔹 May provide modernized feature sets

## 4. Refactor / Re-architect
Driven by a strong business need to **add features, scale, or performance**, which would otherwise be difficult to achieve in the application’s current environment.

> 🔹 Example: Move monolith to microservices  
> 🔹 High effort, high reward  
> 🔹 Cloud-native transformation

## 5. Retire
Identify IT assets that are **no longer useful or in use**. Retiring them helps streamline operations and boost the business case for cloud adoption.

> 🔹 Reduces technical debt  
> 🔹 Frees up budget and resources  
> 🔹 Cleans up legacy environment

## 6. Retain
You may want to **keep certain applications on-premises** because they are not ready for migration or recently upgraded.

> 🔹 Example: Apps under active vendor contracts  
> 🔹 Useful for phased migrations  
> 🔹 “Revisit later” strategy

Mnemonic to Remember
“If it’s RAC, then use EC2 back.”
 If your app needs Oracle RAC, you must go back to EC2 instead of using RDS.
And for backups:
“DLM wins over scripts.”
 Let AWS Data Lifecycle Manager handle snapshots — more reliable, less error-prone, policy-driven.
“Traffic spike? Use CloudFront’s edge to fight.”
 CloudFront buys you time, speed, and global scale, immediately.
“WAF at the edge stops the wedge.”
 Deploy AWS WAF at CloudFront to block exploits right at the edge — before they hit your app.