# SAA-C03 Exam Cram Sheet

Oct 22, 2026 · @Simão

## 1. Amazon S3 Security

Know the four encryption options cold, plus Object Lock modes and when to use pre-signed URLs.

**Concepts**

- **Amazon S3**: object storage — files (objects) stored in buckets, virtually unlimited, 11 nines of durability. Used for backups, static websites, media, data lakes.
- **Encryption at rest / in transit**: at rest = data encrypted on disk so stolen storage is unreadable; in transit = TLS/HTTPS protects data moving over the network.
- **Bucket policy**: JSON resource-based policy attached to a bucket saying who can do what. Used for cross-account access, public access, enforcing encryption.
- **CORS**: browser rule that blocks a web page from calling another domain unless that domain explicitly allows it.
- **MFA Delete**: requires an MFA code for destructive versioning operations — protects against accidental or malicious deletion.
- **Pre-signed URL**: time-limited link that grants temporary access to a private object without making it public.
- **WORM (Object Lock / Vault Lock)**: Write Once Read Many — data can't be changed or deleted for a set period. Used for regulatory compliance.
- **Access Points / Object Lambda**: named entry points with their own policy to simplify access for many teams / code that transforms objects as they are read.

| Option | Key managed by | Exam cues |
| --- | --- | --- |
| SSE-S3 | AWS (AES-256) | Default for all new objects. Header `x-amz-server-side-encryption: AES256` |
| SSE-KMS | KMS key | Audit key use in CloudTrail, user control. Counts against KMS API quota (throttling at high volume) |
| DSSE-KMS | KMS, two layers | Compliance requiring dual-layer encryption |
| SSE-C | Customer, sent per request | HTTPS mandatory; AWS never stores the key |
| Client-side | Customer | Encrypt before upload, decrypt after download |

- **Force encryption in transit**: bucket policy Deny when `aws:SecureTransport = false`.
- **Force SSE**: bucket policy denying PutObject without the encryption header. Bucket policies are evaluated before default encryption.
- **CORS**: browser fetching from another bucket/origin → enable CORS on the *target* bucket (`Access-Control-Allow-Origin`).
- **MFA Delete**: needs versioning; only the root account can enable it. Required to permanently delete a version or suspend versioning.
- **Access logs**: write to a different bucket in the same region. Never log to the same bucket (infinite loop).
- **Pre-signed URLs**: temporary access with the creator's permissions. Console up to 12 h, CLI/SDK up to 7 days. Use for private downloads/uploads by non-IAM users.
- **Glacier Vault Lock**: WORM; once the policy is locked, nobody can change it.
- **S3 Object Lock** (needs versioning), per object version:
  - *Compliance mode*: nobody, root included, can delete or shorten retention.
  - *Governance mode*: users with special IAM permission can override.
  - *Legal Hold*: indefinite protection, independent of retention (`s3:PutObjectLegalHold`).
- **Access Points**: one per team/prefix, each with its own policy and DNS name; can be VPC-only.
- **S3 Object Lambda**: Lambda transforms objects on GET (redact PII, convert format, resize) without storing a copy.
- **Block Public Access** at account/bucket level overrides any policy that would make data public.

## 2. CloudFront & Global Accelerator

CloudFront caches content at the edge; Global Accelerator routes traffic over the AWS backbone to static anycast IPs with no caching.

**Concepts**

- **CDN (Content Delivery Network)**: network of servers around the world that cache content close to users to reduce latency and offload the origin.
- **CloudFront**: AWS's CDN. Used to speed up websites, APIs, downloads and video, and to shield origins from DDoS.
- **Edge location (PoP)**: small AWS site near users where content is cached and traffic enters the AWS network.
- **Origin**: the source CloudFront fetches content from when it is not cached (S3 bucket, ALB, EC2, any HTTP server).
- **TTL**: how long an object stays cached before CloudFront re-fetches it.
- **Origin Access Control (OAC)**: lets only CloudFront read a private S3 bucket, so users can't bypass the CDN.
- **Global Accelerator**: network-layer service giving your app 2 static **anycast IPs**; users enter AWS at the nearest edge and travel on AWS's private backbone to the closest healthy endpoint. Used to improve performance and availability of global TCP/UDP apps.
- **Anycast IP**: one IP address announced from many locations; each user is routed to the nearest one.

**CloudFront (CDN)**

- Hundreds of edge locations; caches content with a TTL. Built-in Shield Standard; attach WAF.
- Origins: S3 bucket (secure with **Origin Access Control** + bucket policy; OAC replaces OAI), S3 static website, custom HTTP origin (ALB, EC2, any HTTP backend), VPC origins for private ALB/EC2.
- Public ALB/EC2 origin: its security group must allow CloudFront's public IP ranges.
- **Geo restriction**: allowlist or blocklist of countries (e.g. copyright licensing).
- **Price classes**: All, 200, 100 (cheapest regions only) to cut cost.
- **Cache invalidation**: force refresh before TTL expiry (`/*` or specific paths).
- **Signed URLs / signed cookies**: restrict premium/private content (one file vs many files).
- CloudFront vs **S3 Cross-Region Replication**: CloudFront = global edge cache for static content tolerant of TTL; CRR = full copy in a few regions, near real-time, for low-latency reads in those regions.

**Global Accelerator**

- 2 static anycast IPs; traffic enters at the nearest edge and rides the AWS network to the app.
- Endpoints: Elastic IP, EC2, ALB, NLB (public or private), across regions.
- Health checks with failover in under a minute (good for DR). Shield integration.
- Pick it for: non-HTTP (TCP/UDP: gaming, IoT, VoIP), HTTP needing static IPs, deterministic fast regional failover.

| Need | Choose |
| --- | --- |
| Cache static/dynamic HTTP content near users | CloudFront |
| Static IPs for a global app, UDP/TCP, instant regional failover | Global Accelerator |
| Static IP + WAF | Global Accelerator in front of ALB (WAF on ALB) |

## 3. AWS Storage Extras

Match the protocol and the use case to the right service: Snow for offline bulk transfer, FSx for third-party file systems, Storage Gateway for hybrid, DataSync for scheduled sync.

**Concepts**

- **Snow Family**: rugged physical devices AWS ships to you to move TB–PB of data offline (when the network is too slow) or to run compute in places with no/poor connectivity (edge computing).
- **File vs block vs object storage**: file = shared folders over a network (NFS for Linux, SMB for Windows); block = raw disk volumes (EBS, iSCSI); object = files with metadata via API (S3).
- **Amazon FSx**: fully managed, high-performance third-party file systems (Windows, Lustre, NetApp ONTAP, OpenZFS). Used when EFS doesn't fit the OS, protocol or performance needed.
- **HPC**: High Performance Computing — many machines processing large datasets in parallel (simulations, ML training).
- **Storage Gateway**: software appliance (VM or hardware) on-premises that exposes cloud storage to local apps as file shares, disks or tapes. Used for hybrid cloud, backups, tiering to S3.
- **Transfer Family**: managed FTP/FTPS/SFTP server that stores files in S3 or EFS — keeps existing FTP workflows.
- **DataSync**: online transfer service that moves and syncs large amounts of data to/from AWS storage on a schedule.

**Snow Family** (Snowball Edge Storage Optimized / Compute Optimized)

- Offline data migration when a network transfer would take more than \~1 week; also edge computing in disconnected sites.
- Snowball cannot import straight into Glacier: import to S3, then a lifecycle policy to Glacier.
- Managed with AWS OpsHub.

**Amazon FSx**

| Flavour | Protocol / OS | Exam cues |
| --- | --- | --- |
| FSx for Windows File Server | SMB, NTFS | Active Directory, ACLs, DFS; Multi-AZ; mountable from Linux too |
| FSx for Lustre | Lustre (Linux) | HPC, ML, video; reads/writes S3 seamlessly. Scratch = temporary, fast, no replication; Persistent = long-term, replicated in AZ |
| FSx for NetApp ONTAP | NFS, SMB, iSCSI | Move ONTAP/NAS workloads; Linux, Windows, macOS; instant point-in-time cloning, dedup/compression |
| FSx for OpenZFS | NFS | Move ZFS workloads; up to 1M IOPS, cloning |

**Storage Gateway** (hybrid: on-prem to S3/Glacier/EBS)

- **S3 File Gateway**: NFS/SMB → S3 objects; caches most-recent data locally; AD auth for SMB; lifecycle to Glacier.
- **FSx File Gateway**: local cache for FSx for Windows (closed to new customers).
- **Volume Gateway**: iSCSI block volumes backed by EBS snapshots. *Cached* (data in S3, hot data local) vs *Stored* (all data local, async backup to S3).
- **Tape Gateway**: virtual tape library (iSCSI) for existing backup software → S3 / Glacier.
- Hardware appliance available when no virtualization on-prem.

**Other transfer services**

- **AWS Transfer Family**: managed FTP/FTPS/SFTP into S3 or EFS.
- **DataSync**: scheduled (not continuous) sync on-prem/other cloud → S3, EFS, FSx (needs agent), or AWS ↔ AWS (no agent). Preserves file permissions and metadata.

**Quick map**: S3 = object · Glacier = archive · EBS = one instance, one AZ (multi-attach io1/io2) · Instance Store = max IOPS, ephemeral · EFS = Linux NFS, multi-AZ · FSx = Windows/Lustre/ONTAP/ZFS · Storage Gateway = hybrid · Transfer Family = FTP to S3/EFS · DataSync = scheduled sync · Snow = physical transfer.

## 4. Decoupling: SQS, SNS, Kinesis, Amazon MQ

SQS = queue (pull), SNS = pub/sub (push), Kinesis = real-time streams with replay, Amazon MQ = open protocols for lift-and-shift.

**Concepts**

- **Decoupling**: putting a messaging layer between components so each can scale, fail and be deployed independently. Synchronous = direct call and wait; asynchronous = drop a message and move on.
- **Amazon SQS**: fully managed message queue. Producers send messages, consumers poll and delete them. Used to buffer work and absorb traffic spikes.
- **Amazon SNS**: fully managed pub/sub. A publisher sends one message to a topic; it is pushed to every subscriber. Used for notifications and fan-out.
- **Kinesis Data Streams**: ingest and store real-time streaming data (clickstreams, logs, IoT telemetry) for custom consumers, with replay.
- **Amazon Data Firehose**: loads streaming data into storage/analytics destinations automatically, no code or capacity to manage.
- **Amazon MQ**: managed message broker (ActiveMQ/RabbitMQ) speaking industry-standard protocols. Used to migrate existing apps without rewriting their messaging.

**SQS**

- Standard: unlimited throughput, at-least-once delivery, best-effort ordering. Retention 4 days default, max 14 days. Message size 256 KB (raised to 1 MiB in 2025; exam may still say 256 KB).
- **Visibility timeout** (default 30 s): message hidden while being processed; consumer extends with `ChangeMessageVisibility`. Too short = duplicates.
- **Long polling** (1–20 s): fewer API calls, lower cost and latency.
- **Dead-letter queue**: after `MaxReceiveCount` failures.
- **Delay queue**: up to 15 min.
- **FIFO**: strict ordering + exactly-once (dedup ID); 300 msg/s, 3,000 with batching (higher in high-throughput mode); name ends in `.fifo`; ordering per **Message Group ID**.
- Scale an ASG on the CloudWatch metric `ApproximateNumberOfMessages` (queue length).
- Buffer writes to a DB: app → SQS → workers → DB (absorbs spikes, no lost writes).
- Security: HTTPS in flight, KMS at rest, SQS access policy for cross-account and for S3/SNS to write.

**SNS**

- Pub/sub push: one message → many subscribers (SQS, Lambda, Firehose, HTTP(S), email, SMS, mobile push).
- **Fan-out** = SNS + multiple SQS queues: durable, retryable, add subscribers later. Use for an S3 event to many queues (one event rule per prefix/type otherwise).
- **SNS FIFO** → SQS FIFO for ordered fan-out. **Message filtering**: JSON filter policy per subscription.

**Kinesis**

- **Kinesis Data Streams**: real-time, retention 1–365 days, replay, ordering per partition key (shard). Provisioned (per shard: 1 MB/s or 1,000 rec/s in, 2 MB/s out) or On-demand. You write consumers (Lambda, KCL apps, Flink, Firehose).
- **Amazon Data Firehose** (ex Kinesis Data Firehose): fully managed, auto-scaling, **near real-time** (buffering), no storage/replay. Destinations: S3, Redshift (via S3 COPY), OpenSearch, Splunk, partners, HTTP. Optional Lambda transformation; failed records to S3.

|  | SQS | SNS | Kinesis Data Streams |
| --- | --- | --- | --- |
| Model | Consumers pull | Pushed to subscribers | Consumers pull (or enhanced fan-out push) |
| After read | Deleted | Not persisted | Kept; replayable |
| Ordering | FIFO queues | FIFO topics | Per shard |
| Scaling | Automatic | Automatic | Shards / on-demand |

**Amazon MQ**: managed ActiveMQ / RabbitMQ. Pick when migrating on-prem apps that use MQTT, AMQP, STOMP, OpenWire, WSS without rewriting to SQS/SNS. Less scalable; HA = active/standby across 2 AZs (shared EFS storage for ActiveMQ).

## 5. Containers: ECS, Fargate, ECR, EKS

ECS is AWS's own orchestrator, EKS is managed Kubernetes, Fargate removes the servers from either, ECR stores the images.

**Concepts**

- **Container**: lightweight package of an app plus its dependencies (Docker image) that runs the same on any host. Starts in seconds, uses less overhead than a VM.
- **Orchestrator**: system that places containers on servers, scales them, restarts failed ones and connects them to load balancers.
- **Amazon ECS**: AWS's own container orchestrator. Used to run Docker workloads simply, deeply integrated with AWS.
- **Task definition / Task / Service**: blueprint (image, CPU, memory, ports, IAM role) / a running copy of it / keeps N tasks running behind a load balancer.
- **AWS Fargate**: serverless compute for containers — no EC2 instances to manage; pay per task CPU/memory.
- **Amazon ECR**: managed Docker image registry to store and pull container images.
- **Amazon EKS**: managed Kubernetes. Used when you already use Kubernetes or want portability across clouds.

**ECS**

- **EC2 launch type**: you provision and manage the EC2 instances (ECS agent runs on each).
- **Fargate launch type**: serverless; define CPU/RAM in the task definition, no instances to manage.
- **IAM roles**: *EC2 Instance Profile* = used by the ECS agent (ECS API, CloudWatch Logs, pull from ECR, read Secrets/SSM). *Task Role* = per task, what the app can call (e.g. S3, DynamoDB).
- **Load balancing**: ALB (most cases), NLB (high throughput / PrivateLink).
- **Shared storage**: EFS works with both EC2 and Fargate (multi-AZ, persistent). S3 cannot be mounted.
- **Service Auto Scaling** (Application Auto Scaling): target tracking on CPU, memory or ALB requests per target; step; scheduled. For EC2 launch type also scale the instances → **ECS Cluster Capacity Provider** (preferred over plain ASG).
- Patterns: S3 upload → EventBridge → run ECS task; EventBridge schedule → task every hour; tasks poll SQS; task stopped events → EventBridge → SNS alert.

**ECR**: private and public image registry, stored in S3; IAM access; image scanning, versioning, tags, lifecycle policies.

**EKS**

- Managed Kubernetes control plane; pick when already on Kubernetes or wanting cloud-agnostic portability.
- Node types: Managed Node Groups, Self-Managed Nodes, Fargate.
- Storage via StorageClass + CSI driver: EBS, EFS (only one that works with Fargate), FSx for Lustre, FSx for NetApp ONTAP.

**Also know**

- **App Runner**: deploy a web app/API from source or container image with zero infra config; auto-scaling, HA, load balancer, HTTPS built in.
- **App2Container**: CLI to containerize legacy Java/.NET apps and generate ECS/EKS artifacts.

## 6. Serverless Overview

The exam tests Lambda limits, DynamoDB features, API Gateway endpoint types and the Cognito User Pool vs Identity Pool split.

**Concepts**

- **Serverless**: you don't provision or manage servers; the service scales automatically and you pay per use. Covers Lambda, DynamoDB, API Gateway, S3, SQS, SNS, Fargate, Cognito, etc.
- **AWS Lambda**: runs your code (a function) in response to events (HTTP call, S3 upload, queue message, schedule). Pay per request and per ms of run time.
- **Cold start**: delay when Lambda creates a new execution environment and runs init code before the first request.
- **Amazon DynamoDB**: serverless NoSQL key-value/document database with single-digit millisecond latency at any scale.
- **Amazon API Gateway**: managed "front door" to create, secure, throttle and publish REST and WebSocket APIs, usually backed by Lambda.
- **AWS Step Functions**: orchestrates multiple services into a workflow (state machine) with retries, branches and waits.
- **Amazon Cognito**: user identity for web/mobile apps — sign-up/sign-in and temporary AWS credentials for app users.
- **Lambda@Edge / CloudFront Functions**: small functions run at CloudFront edge locations to customize requests/responses close to users.

**Lambda limits (per region/function)**

| Limit | Value |
| --- | --- |
| Memory | 128 MB – 10 GB (more memory = more vCPU) |
| Max timeout | 15 min (longer → ECS/Fargate, Step Functions, Batch) |
| /tmp storage | 512 MB – 10 GB |
| Env variables | 4 KB |
| Concurrency | 1,000 default (soft limit) |
| Package | 50 MB zipped / 250 MB unzipped; container image up to 10 GB |

- **Reserved concurrency** caps one function; **Provisioned concurrency** pre-warms to avoid cold starts. **SnapStart** cuts cold starts (Java, Python, .NET).
- **Lambda in a VPC** gets an ENI. Many Lambdas → RDS = too many connections → **RDS Proxy** (pools connections, IAM auth, Secrets Manager; never public).
- RDS/Aurora (MySQL, PostgreSQL) can invoke Lambda from inside the DB on data events. RDS Event Notifications only cover the DB instance (not data changes).
- **CloudFront Functions** (JS, sub-ms, viewer request/response, millions of req/s: header rewrites, URL redirects, JWT checks) vs **Lambda@Edge** (Node/Python, up to 5–10 s, all 4 triggers, network/body access, 3rd-party libs).

**DynamoDB**

- Serverless NoSQL, multi-AZ, item max 400 KB. Capacity: Provisioned (RCU/WCU + auto scaling) or On-demand (unpredictable traffic, pay per request).
- **DAX**: microsecond read cache, no code change. ElastiCache instead for aggregated/computed results.
- **Streams** (24 h retention, → Lambda) vs **Kinesis Data Streams** (up to 1 year, more consumers).
- **Global Tables**: active-active multi-region, needs Streams. **TTL** to expire items.
- Backups: PITR (up to 35 days), on-demand backups; export to S3 (needs PITR) / import from S3, no capacity consumed.

**API Gateway**

- Endpoint types: **Edge-optimized** (default, via CloudFront), **Regional**, **Private** (VPC endpoint + resource policy).
- Integration timeout 29 s by default. REST and WebSocket APIs; caching, throttling, usage plans with API keys.
- Auth: IAM, Cognito User Pools, Lambda authorizer. Custom domain cert in ACM: **us-east-1** for edge-optimized, same region for regional.

**Step Functions**: serverless visual workflows orchestrating Lambda, ECS, Batch, DynamoDB, SQS, etc.; supports human approval steps.

**Cognito**

- **User Pools**: sign-in/sign-up for app users, returns JWT; federation with Google/Facebook/SAML; integrates with API Gateway and ALB.
- **Identity Pools**: exchange a token for **temporary AWS credentials** so users access S3/DynamoDB directly; row-level security via IAM policy variables (`${cognito-identity.amazonaws.com:sub}`).

## 7. Serverless Solution Architectures

The exam reuses a handful of patterns; recognise the scenario and pick the matching building block.

**Concepts**

- **Solution architecture**: combining AWS services to meet a scenario's requirements — scalability, availability, cost, security, minimal ops.
- **Microservices**: an app split into small, independently deployable services that talk over APIs or messages.
- **Event-driven architecture**: components react to events (upload, insert, message) instead of calling each other directly — loose coupling.
- **Amazon SES**: managed service to send (and receive) email at scale.
- **AWS IoT Core**: connects IoT devices to AWS and routes their messages to other services.

| Scenario | Pattern |
| --- | --- |
| Mobile app: REST API + per-user data | API Gateway → Lambda → DynamoDB; Cognito User Pools for auth |
| Users upload/download their own files directly | Cognito Identity Pool → temp creds → S3 (policy scoped to user prefix) |
| Read-heavy DynamoDB, high read cost | DAX in front of DynamoDB; API Gateway response caching for static responses |
| Global static website | S3 + CloudFront with OAC |
| Global low-latency DB writes/reads | DynamoDB Global Tables |
| Welcome email on new user | DynamoDB Streams → Lambda → SES |
| Thumbnails on upload | S3 event → Lambda (or SQS/SNS) → thumbnail bucket |
| Microservices, sync calls | API Gateway or ALB in front of each service |
| Microservices, async | SQS, SNS, Kinesis, EventBridge between services |
| Distribute large static files (software updates) cheaply from EC2 | Put CloudFront in front of the existing ALB/ASG: caches at edge, no app change, cheaper egress |
| Serverless big-data ingestion | IoT Core → Kinesis Data Streams → Firehose (+ Lambda transform) → S3 → Athena → S3 reports → QuickSight / Redshift |
| Long multi-step workflow > 15 min | Step Functions (or ECS/Fargate tasks) |

Cost/scale rules of thumb: stateless compute (Lambda/Fargate) + managed state (DynamoDB, S3, Aurora Serverless) + queues to absorb bursts.

## 8. Databases in AWS

Most database questions are "which engine fits this data model and access pattern"; the keyword in the question usually names it.

**Concepts**

- **Relational (SQL) database**: data in tables with a fixed schema, joins and ACID transactions.
- **NoSQL database**: flexible schema, scales horizontally; families are key-value, document, graph, wide-column, time-series.
- **OLTP vs OLAP**: OLTP = many small, fast transactions (orders, logins); OLAP = large analytical queries over lots of data (reports).
- **Managed database**: AWS handles provisioning, patching, backups, monitoring and failover; you handle schema and queries.
- **Amazon RDS**: managed relational database for common engines.
- **Amazon Aurora**: AWS-built cloud-native relational engine, MySQL/PostgreSQL compatible, faster and more available than standard RDS.
- **Multi-AZ vs Read Replica**: Multi-AZ = standby copy in another AZ for automatic failover (availability); Read Replica = readable copy to spread read traffic (performance).
- **Cache (ElastiCache)**: in-memory store holding frequently read data to cut latency and database load.

| Keyword in question | Service | Key facts |
| --- | --- | --- |
| Relational, OLTP, SQL, joins | **RDS** (PostgreSQL, MySQL, MariaDB, Oracle, SQL Server, Db2) | Multi-AZ = HA (sync standby, not readable); Read Replicas = read scaling (async, up to 15, cross-region); storage auto scaling; PITR up to 35 days |
| Need OS/DB customization (Oracle, SQL Server) | **RDS Custom** | Access to underlying instance |
| High-performance relational, MySQL/PostgreSQL compatible | **Aurora** | 6 copies across 3 AZs; up to 15 replicas, reader endpoint; Serverless v2 for spiky load; **Global Database** (< 1 s cross-region lag, RTO < 1 min); cloning for test envs |
| Cache, sessions, sub-ms reads | **ElastiCache** (Redis/Valkey, Memcached) | Requires app code changes; Redis = HA, persistence, sorted sets (leaderboards); Memcached = simple, multi-threaded, no replication |
| Durable in-memory primary DB, Redis API | **MemoryDB** | Multi-AZ transaction log |
| Key-value NoSQL, serverless, any scale, ms latency | **DynamoDB** | See section 6 |
| Objects, static files, data lake | **S3** |  |
| MongoDB-compatible documents | **DocumentDB** | Aurora-like storage |
| Graph, relationships (social, fraud) | **Neptune** | Neptune Streams for change capture |
| Apache Cassandra, CQL | **Keyspaces** | Serverless |
| Time-series (IoT, metrics) | **Timestream** | Serverless, recent data in memory |
| Immutable, cryptographically verifiable ledger | **QLDB** | Discontinued 2025, but may still appear in questions |
| Search, partial-text match | **OpenSearch** | See section 9 |

- **RDS/Aurora security**: KMS at rest (set at creation; to encrypt an unencrypted DB: snapshot → copy encrypted → restore), TLS in flight, IAM DB auth (MySQL/PostgreSQL), security groups, no SSH except RDS Custom.
- Read Replica vs Multi-AZ: replicas scale reads, Multi-AZ gives failover. A replica can be promoted (DR).
- Lots of Lambda connections → **RDS Proxy** (also reduces failover time \~66%).

## 9. Data & Analytics

Athena queries S3 in place, Redshift is the warehouse, Glue does ETL and the catalog, Lake Formation governs the lake.

**Concepts**

- **Data lake**: central store (usually S3) holding raw data of any format and scale, analysed later.
- **Data warehouse**: structured store optimised for analytical (OLAP) SQL queries across large datasets — Redshift.
- **ETL**: Extract, Transform, Load — moving data from sources, cleaning/reshaping it, and loading it for analysis.
- **Columnar format (Parquet/ORC)**: stores data by column, so queries read only the columns they need — faster and cheaper.
- **Data catalog**: metadata registry of datasets (tables, schemas, locations) that query engines use to find data.
- **Big data processing**: distributed processing of huge datasets across a cluster (Hadoop, Spark on EMR).
- **Stream processing**: analysing data continuously as it arrives (Flink, Kafka/MSK) rather than in batches.
- **BI (Business Intelligence)**: dashboards and reports for business users — QuickSight.

* **Athena**: serverless SQL on S3, \~$5/TB scanned. Lower cost/faster: columnar formats (**Parquet/ORC**, convert with Glue), compression, partitioning (`s3://bucket/year=/month=/`), larger files. **Federated Query** via Lambda connectors (RDS, DynamoDB, on-prem) with results in S3.
* **Redshift**: OLAP, columnar, MPP data warehouse (leader + compute nodes), provisioned or Serverless. Load via S3 COPY (Enhanced VPC Routing keeps it off the internet) or Firehose. Snapshots incremental to S3; automated cross-region snapshot copy for DR (KMS-encrypted needs a snapshot copy grant). **Redshift Spectrum** queries S3 data without loading (cluster required).
* **OpenSearch** (ex Elasticsearch): search any field, partial matches. Common pair: DynamoDB → Streams → Lambda → OpenSearch for search; ingest via Firehose or CloudWatch Logs subscription.
* **EMR**: managed Hadoop/Spark clusters on EC2 for big-data processing. Master, Core (store data, On-Demand/Reserved), Task (compute only, use **Spot**).
* **QuickSight**: serverless BI dashboards; SPICE in-memory engine; column-level security (Enterprise); users/groups are QuickSight-only (not IAM).
* **Glue**: serverless ETL (e.g. CSV → Parquet); **Data Catalog** filled by crawlers, used by Athena, Redshift Spectrum, EMR. Job Bookmarks avoid reprocessing; DataBrew for no-code cleaning; Glue Streaming ETL.
* **Lake Formation**: build a data lake in days; centralized fine-grained access (row/column level) applied across Athena, Redshift, EMR; built on Glue.
* **Managed Service for Apache Flink** (ex Kinesis Data Analytics): real-time stream processing from Kinesis Data Streams or MSK (not from Firehose).
* **MSK**: managed Apache Kafka (or MSK Serverless).

|  | Kinesis Data Streams | Amazon MSK |
| --- | --- | --- |
| Message size | 1 MB | 1 MB default, configurable higher (e.g. 10 MB) |
| Unit | Shards (split/merge) | Topics + partitions (add only) |
| Retention | Up to 365 days | As long as you pay for EBS storage |
| Encryption in flight | TLS | Plaintext or TLS |

## 10. Machine Learning

Only service-to-use-case mapping is tested; no ML theory.

**Concepts**

- **AI services**: pre-trained models exposed as APIs (vision, speech, language, documents) — no ML expertise or training needed. Used to add intelligence to apps quickly.
- **Custom ML (SageMaker)**: platform where data scientists build, train, tune and deploy their own models when no ready-made service fits.
- **NLP**: Natural Language Processing — extracting meaning from text (sentiment, entities). **ASR / TTS**: speech → text / text → speech.

| Service | Does | Typical question cue |
| --- | --- | --- |
| Rekognition | Image/video analysis: objects, faces, text, celebrity, content moderation | "detect inappropriate images", "face search" |
| Transcribe | Speech → text; PII redaction; auto language detection | "subtitles", "call transcripts" |
| Polly | Text → speech; lexicons, SSML | "read articles aloud" |
| Translate | Language translation | "localize content" |
| Lex | Chatbots (speech recognition + NLU); with **Connect** = cloud contact center | "chatbot", "call center" |
| Comprehend | NLP: sentiment, entities, key phrases, topics | "analyze customer reviews" |
| Comprehend Medical | Extract PHI/medical info from clinical text | "doctor notes" |
| SageMaker | Build, train and deploy custom ML models | "data scientists build a model" |
| Kendra | ML-powered document search (natural language) | "search internal documents" |
| Personalize | Real-time recommendations | "recommended products" |
| Textract | Extract text, forms, tables from scanned docs | "read invoices / ID documents" |

## 11. Monitoring & Audit: CloudWatch, CloudTrail, Config

CloudWatch = performance and logs, CloudTrail = who called which API, Config = how resource configuration changed and whether it is compliant.

**Concepts**

- **Monitoring**: collecting metrics and logs to see how systems perform and to react to problems. **Auditing**: recording who did what, when, for security and compliance.
- **Amazon CloudWatch**: AWS's monitoring service — metrics, logs, alarms, dashboards.
- **Metric**: a time-ordered numeric value (CPUUtilization, request count). **Alarm**: watches a metric against a threshold and triggers an action. **Log group**: container for logs of one app/service.
- **Amazon EventBridge**: serverless event bus that routes events (from AWS services, SaaS, your apps, or a schedule) to targets.
- **AWS CloudTrail**: records every API call in the account (who, what, when, from where). Used for audits and security investigations.
- **AWS Config**: inventory of resources plus their configuration history, evaluated against compliance rules.

**CloudWatch**

- **Metrics**: EC2 default every 5 min, detailed monitoring 1 min. **RAM, disk space, processes are not native** → install the **unified CloudWatch Agent** (also ships logs). Custom metrics via `PutMetricData`.
- **Metric Streams**: near real-time to Firehose (→ S3, Redshift, OpenSearch) or partners (Datadog, Splunk).
- **Logs**: log groups/streams; retention 1 day–10 years or never expire. Export to S3 with `CreateExportTask` (not real-time, up to 12 h). Real-time → **subscription filters** to Kinesis Data Streams, Firehose, Lambda; cross-account aggregation. **Logs Insights** = query language for logs.
- **Alarms**: states OK / INSUFFICIENT\_DATA / ALARM. Actions: EC2 (stop, terminate, reboot, **recover** — same IP, metadata, placement), Auto Scaling, SNS. **Composite alarms** (AND/OR) reduce noise. Test with `set-alarm-state`.
- Insights: Container Insights (ECS/EKS/Fargate), Lambda Insights, Contributor Insights (top-N talkers), Application Insights.

**EventBridge** (ex CloudWatch Events): cron schedules or event patterns → targets (Lambda, SQS, SNS, ECS task, Step Functions…). Default, partner (SaaS) and custom buses; archive & replay; schema registry; resource policy for cross-account central bus.

**CloudTrail**

- API call history for governance/audit; on by default, 90 days of events. Trail → S3 / CloudWatch Logs for longer retention (query with Athena). Apply to all regions or one.
- **Management events** (logged by default), **Data events** (not by default: S3 object-level, Lambda invoke), **CloudTrail Insights** (unusual write activity).
- React to an API call (e.g. `DeleteTable`, root login) → CloudTrail → EventBridge → SNS.

**AWS Config**

- Records configuration changes and compliance over time, per region (aggregate across regions/accounts).
- Managed rules (e.g. "SSH open to 0.0.0.0/0", "S3 bucket public") and custom rules (Lambda). Config **does not prevent** actions.
- **Remediation** via SSM Automation documents (auto-retry); notifications via EventBridge or SNS.

| Question asks | Answer |
| --- | --- |
| Latency / CPU / errors over time | CloudWatch |
| Who deleted / changed this resource | CloudTrail |
| Was this resource's config compliant last week? Auto-fix non-compliance | Config |

## 12. IAM Advanced

Expect Organizations/SCP questions, policy evaluation, resource-based policy vs role, and which identity service fits.

**Concepts**

- **IAM**: controls *who* can access AWS (authentication) and *what* they can do on *which* resources (authorization).
- **Policy**: JSON document of Allow/Deny permissions. *Identity-based* = attached to a user/group/role; *resource-based* = attached to a resource (S3 bucket, SQS queue, KMS key).
- **Role**: identity with temporary credentials that users, services or other accounts assume. Used instead of long-lived access keys.
- **AWS Organizations**: manage many AWS accounts centrally — structure, billing, guardrails.
- **SCP (Service Control Policy)**: Organization-level policy that sets the maximum permissions for accounts.
- **Permission boundary**: maximum permissions an individual user/role can ever have.
- **SSO / Federation**: log in once with an existing identity (corporate directory, Google) to access many accounts/apps.
- **Active Directory (AD)**: Microsoft directory of users, computers and groups used for enterprise authentication.

**AWS Organizations**

- Management account + member accounts grouped in OUs. Consolidated billing, aggregated volume discounts, shared Reserved Instance / Savings Plans discounts.
- **SCPs**: guardrails on OUs/accounts; restrict even the account root. **Do not apply to the management account.** Nothing is allowed unless also allowed by IAM (SCP never grants). Deny-list or allow-list strategy.
- Tag policies to standardize tags across accounts.

**Policy details**

- Useful conditions: `aws:SourceIp`, `aws:RequestedRegion`, `aws:PrincipalOrgID` (restrict a resource to your Organization), `aws:MultiFactorAuthPresent`, `ec2:ResourceTag`.
- S3 ARNs: `s3:ListBucket` → `arn:aws:s3:::bucket`; `s3:GetObject/PutObject` → `arn:aws:s3:::bucket/*`.
- **Evaluation**: explicit Deny wins → SCP must allow → resource policy → permission boundary → session policy → identity policy. Default is implicit deny.
- **Permission boundaries**: max permissions for a user or role (not groups). Effective = identity policy ∩ boundary ∩ SCP. Lets admins delegate safely.
- **Resource-based policy vs assume role**: assuming a role means giving up your original permissions; a resource-based policy lets the principal keep them (e.g. copy from cross-account bucket into own DynamoDB).
- EventBridge targets: Lambda, SNS, SQS, S3, API Gateway use resource-based policies; Kinesis, ECS, Step Functions, SSM need an IAM role.

**Identity services**

- **IAM Identity Center** (ex AWS SSO): one login for all accounts in the Organization + SAML business apps; permission sets; identities from built-in store, AD, or external IdP (Okta, Entra ID); ABAC.
- **Directory Service**: *Managed Microsoft AD* (real AD in AWS, trust with on-prem, MFA); *AD Connector* (proxy to on-prem AD, no caching); *Simple AD* (standalone AD-compatible, no on-prem link).
- **Control Tower**: sets up a governed multi-account landing zone on Organizations; **guardrails** = preventive (SCPs) and detective (Config); Account Factory for new accounts; compliance dashboard.

## 13. Security & Encryption: KMS, SSM Parameter Store, Shield, WAF

KMS keys are regional and governed by key policies; WAF is layer 7, Shield is DDoS; know which security service detects what.

**Concepts**

- **Encryption in flight / at rest / client-side**: TLS on the network / encrypted on the server's storage / encrypted by the client before sending, so the server never sees plaintext.
- **AWS KMS**: managed service to create and control encryption keys; integrated with most AWS services, every key use logged in CloudTrail. KMS encrypts up to 4 KB directly; larger data uses **envelope encryption** (a data key that KMS encrypts).
- **SSM Parameter Store**: secure, hierarchical store for configuration values and secrets.
- **AWS Secrets Manager**: stores secrets (DB passwords, API keys) and rotates them automatically.
- **AWS Certificate Manager (ACM)**: provisions and renews TLS/SSL certificates for HTTPS.
- **DDoS attack**: flood of traffic from many sources meant to make a service unavailable.
- **AWS Shield**: DDoS protection service.
- **AWS WAF**: web application firewall — filters HTTP(S) requests by rules (IPs, SQL injection, XSS, rate limits).
- **Firewall Manager**: applies security rules centrally across all accounts in an Organization.

**KMS**

- Key types: **AWS owned** (e.g. SSE-S3, free, invisible), **AWS managed** (`aws/s3`, `aws/ebs`; auto-rotated yearly), **Customer managed** (you control policy; optional automatic rotation; imported key material = manual rotation only).
- **Symmetric** (AES-256, never leaves KMS, used by AWS services) vs **asymmetric** (RSA/ECC; public key downloadable for encryption/signing outside AWS).
- **Key policy** controls access; default policy trusts the account's IAM. Cross-account use needs the key policy to allow the other account.
- Keys are regional: copying an encrypted snapshot/AMI to another region **re-encrypts with a key in the destination region**.
- **Multi-Region Keys**: same key ID/material replicated across regions; for client-side encryption of globally replicated data (DynamoDB Global Tables, Aurora Global). Not global keys — each is managed independently.
- **Share encrypted AMI/snapshot cross-account**: share the AMI/snapshot + allow the target account in the KMS key policy (Decrypt, DescribeKey, CreateGrant, ReEncrypt).
- S3 replication: unencrypted and SSE-S3 objects replicate by default; SSE-KMS needs the option enabled + destination key + permissions.

**Secrets**

- **SSM Parameter Store**: config and secrets in a hierarchy, versioned, KMS for SecureString. Standard tier (free, 4 KB) vs Advanced (8 KB, parameter policies e.g. expiration/TTL).
- **Secrets Manager**: **automatic rotation** via Lambda, native RDS/Aurora/Redshift integration, multi-region replication. Pick it whenever the question says rotate.
- **ACM**: free public TLS certs, auto-renewal for ACM-issued certs; attach to ELB, CloudFront, API Gateway (not directly to EC2). Imported certs don't auto-renew — monitor expiry with Config rule / EventBridge (alerts from 45 days).

**Edge protection**

- **WAF**: layer 7 on ALB, API Gateway, CloudFront, AppSync, Cognito User Pool (**not NLB**). Web ACL rules: IP sets, headers/body, geo-match, size, SQL injection, XSS, **rate-based rules** (HTTP floods). Managed rule groups.
- **Shield Standard**: free, automatic L3/L4 protection (SYN/UDP floods, reflection). **Shield Advanced**: \~$3,000/month per org, 24/7 Shield Response Team, cost protection for scaling during attacks, automatic app-layer mitigation via WAF.
- **Firewall Manager**: central management across the Organization of WAF, Shield Advanced, security groups, Network Firewall, Route 53 Resolver DNS Firewall; auto-applies to new resources.

**Detection**

| Service | Finds |
| --- | --- |
| GuardDuty | Threats via ML on CloudTrail, VPC Flow Logs, DNS logs (+ EKS, S3, RDS, Lambda, EBS malware). Crypto-mining. Findings → EventBridge |
| Inspector | Vulnerabilities in EC2 (via SSM agent), ECR images, Lambda; network reachability |
| Macie | Sensitive data / PII in S3 |
| Security Hub | Central security findings dashboard across accounts |
| Detective | Root-cause investigation of findings |

## 14. Networking – VPC

VPC is the heaviest-weighted area: know SG vs NACL, NAT, endpoints, peering vs Transit Gateway, and VPN vs Direct Connect.

**Concepts**

- **VPC (Virtual Private Cloud)**: your own logically isolated private network inside an AWS region. You choose its IP range, split it into subnets, and control routing and firewalls. Used to run resources privately, decide what is exposed to the internet, and connect securely to on-premises networks.
- **CIDR**: notation for an IP range — `10.0.0.0/16` = 65,536 addresses; the bigger the suffix, the smaller the range.
- **Subnet**: slice of the VPC's IP range inside one AZ. *Public* = has a route to an Internet Gateway; *private* = no direct internet route.
- **Route table**: rules saying where traffic from a subnet goes (local, IGW, NAT, peering, endpoint, VPN).
- **Internet Gateway (IGW)**: lets resources in public subnets reach and be reached from the internet.
- **NAT Gateway**: lets instances in private subnets reach the internet (updates, APIs) while blocking inbound connections.
- **Security Group / NACL**: virtual firewalls at the instance level / subnet level.
- **VPC Peering / Transit Gateway**: connect two VPCs directly / connect many VPCs and on-prem networks through a central hub.
- **VPC Endpoint / PrivateLink**: reach AWS services or another VPC's service over AWS's private network, without going through the internet.
- **Site-to-Site VPN / Direct Connect**: encrypted tunnel over the internet to on-prem / dedicated private physical connection to AWS.
- **VPC Flow Logs**: record of IP traffic metadata in the VPC, used for troubleshooting and security analysis.

**Basics**

- VPC CIDR /16 to /28; private ranges 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16. CIDRs must not overlap with networks you connect to.
- **5 IPs reserved per subnet** (first 4 + last 1). Need 29 hosts → /27 gives only 27 → choose **/26**.
- **Internet Gateway**: one per VPC; public subnet = route `0.0.0.0/0 → IGW`.
- **Bastion host** in public subnet; its SG allows SSH only from your corporate CIDR.
- **NAT Gateway**: managed, in a public subnet with an Elastic IP, per AZ (one per AZ for HA), 5–100 Gbps, no SG. **NAT instance** = legacy, disable source/destination check, you manage SG and failover.
- **IPv6**: all IPv6 addresses are public; **Egress-only Internet Gateway** = outbound-only for IPv6 (NAT equivalent). IPv4 can't be disabled (dual-stack).

|  | Security Group | Network ACL |
| --- | --- | --- |
| Level | Instance / ENI | Subnet |
| Rules | Allow only | Allow and Deny |
| State | Stateful (return traffic auto-allowed) | Stateless (allow ephemeral ports 1024–65535 for responses) |
| Evaluation | All rules | In number order, first match wins |
| Block a specific IP | Not possible | Yes |

**Connecting VPCs and services**

- **VPC Peering**: private link between 2 VPCs (cross-account/region), **not transitive**, no overlapping CIDRs, update route tables on both sides; can reference peer SG in same region.
- **VPC Endpoints** (no IGW/NAT needed): *Gateway* = **S3 and DynamoDB only**, route-table target, free. *Interface* (PrivateLink) = ENI with private IP, most services, paid; needed for access from on-prem over VPN/DX. Enable private DNS.
- **PrivateLink** (endpoint service): expose a service to thousands of VPCs via **NLB** (provider) + ENI (consumer); no peering, scales.
- **Transit Gateway**: hub-and-spoke, **transitive**, thousands of VPCs + VPN + Direct Connect; regional, peer across regions; share via **RAM**; route tables control who talks to whom. Supports **IP multicast** and **ECMP** (aggregate many VPN tunnels for more bandwidth).
- **VPC Flow Logs**: VPC/subnet/ENI traffic metadata → S3, CloudWatch Logs, Firehose; query with Athena.
- **Traffic Mirroring**: copy ENI traffic to security appliances. **AWS Network Firewall**: L3–L7 protection for the whole VPC.

**Hybrid connectivity**

- **Site-to-Site VPN**: Virtual Private Gateway (AWS side) + Customer Gateway (on-prem, public IP or behind NAT-T). Over the internet, encrypted, quick to set up. Enable route propagation; allow ICMP in SG for ping.
- **VPN CloudHub**: hub-and-spoke between multiple on-prem sites via one VGW.
- **Direct Connect**: private dedicated line. Dedicated (1/10/100 Gbps) or Hosted (50 Mbps–10 Gbps). Takes **> 1 month** to provision. **Not encrypted** by default → IPsec VPN over DX (or MACsec). **Direct Connect Gateway** to reach VPCs in many regions.
- DX resiliency: *High* = connections at 2 DX locations; *Maximum* = 2 connections at each of 2 locations. Cheap backup = Site-to-Site VPN.

**Cost rules**: same-AZ private traffic free; cross-AZ \~$0.01/GB each way; use private IPs over public; S3 → CloudFront free; prefer Gateway Endpoint over NAT Gateway for S3/DynamoDB; ingress free, egress paid.

## 15. Disaster Recovery & Migrations

Pick the DR strategy from the RPO/RTO and budget in the question; pick the migration tool from what is being moved.

**Concepts**

- **Disaster Recovery (DR)**: plan and infrastructure to restore service after a major failure (region outage, data loss, ransomware). Types: on-prem → AWS, AWS region → AWS region, or hybrid.
- **Migration**: moving workloads from on-premises or another cloud into AWS.
- **Homogeneous vs heterogeneous migration**: same database engine on both sides vs different engines (needs schema conversion).
- **CDC (Change Data Capture)**: continuously copying ongoing changes from source to target so they stay in sync until cutover.
- **Rehost (lift-and-shift)**: move servers as-is to EC2 without changing the app.
- **AWS DMS**: Database Migration Service — migrates databases to AWS with minimal downtime.
- **AWS MGN**: Application Migration Service — replicates whole servers into AWS for rehosting.
- **AWS Backup**: centralized, policy-driven backup service across AWS services and accounts.

* **RPO** = how much data you can lose (time since last backup). **RTO** = how long you can be down.

| Strategy | What runs in DR region | RPO / RTO | Cost |
| --- | --- | --- | --- |
| Backup & Restore | Nothing; backups in S3/Glacier, AMIs, snapshots | Hours | Lowest |
| Pilot Light | Only the core (e.g. replicated DB); app servers off | Minutes–hours | Low |
| Warm Standby | Full system at reduced size; scale up on failover | Minutes | Medium |
| Multi-Site / Hot Site | Full production capacity, active-active (Route 53 routing) | Near zero | Highest |

DR tips: Route 53 health checks for failover; RDS cross-region read replicas, Aurora Global Database; EBS snapshots + AMIs copied cross-region; S3 CRR; CloudFormation to rebuild; **AWS Elastic Disaster Recovery (DRS)** for continuous block-level replication of servers into AWS.

**Database migration**

- **DMS**: source stays available during migration; homogeneous or heterogeneous; **CDC** for continuous replication; runs on a replication instance (Multi-AZ option).
- **SCT** (Schema Conversion Tool) only when engines differ (e.g. Oracle → PostgreSQL); not needed for same engine.
- RDS MySQL → Aurora MySQL: restore a snapshot into Aurora (downtime) or create an **Aurora Read Replica** and promote (less downtime). External MySQL → **Percona XtraBackup** to S3 → Aurora, or mysqldump, or DMS. PostgreSQL: same options, no Percona.

**Server and data migration**

- **Application Discovery Service**: inventory on-prem servers and dependencies (agentless connector or agent); view in **Migration Hub**.
- **Application Migration Service (MGN)**: lift-and-shift (rehost) with continuous replication, minimal cutover downtime.
- **VM Import/Export**: move VMs to EC2 and back. Amazon Linux 2 image runs on-prem as a VM for testing.
- **AWS Backup**: central, policy-based backups across services (EC2, EBS, RDS, Aurora, DynamoDB, EFS, FSx, S3, Storage Gateway); cross-region and cross-account copies; **Backup Vault Lock** = WORM, even root can't delete.
- **VMware Cloud on AWS**: extend/migrate vSphere workloads without re-platforming.

**Moving a large dataset** (e.g. 200 TB):

| Method | Rough time |
| --- | --- |
| Internet / VPN at 100 Mbps | \~185 days |
| Direct Connect 1 Gbps | \~18.5 days (plus > 1 month setup) |
| Snowball (several devices in parallel) | \~1 week end to end |

Ongoing replication after the bulk load: Site-to-Site VPN or Direct Connect with DMS or DataSync.
