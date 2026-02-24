# ERP-DBaaS Video Training Scripts

## Document Control

| Field             | Value                              |
|-------------------|------------------------------------|
| Document Title    | ERP-DBaaS Video Training Scripts   |
| Version           | 1.0.0                             |
| Date              | 2026-02-24                         |
| Audience          | Training Content Producers         |
| Classification    | Internal                           |

---

## Video Series Overview

| #  | Title                        | Duration | Audience          | Prerequisites    |
|----|------------------------------|----------|-------------------|------------------|
| 1  | Platform Overview            | 8 min    | All users          | None             |
| 2  | Provisioning a Database      | 12 min   | Operators, Admins  | Video 1          |
| 3  | Scaling and High Availability| 10 min   | Operators, Admins  | Videos 1-2       |
| 4  | Backup and Restore           | 12 min   | Operators, Admins  | Videos 1-2       |
| 5  | Engine Catalog Deep Dive     | 10 min   | All users          | Video 1          |
| 6  | Plugin Management            | 8 min    | Admins             | Videos 1-2       |
| 7  | Credential Rotation          | 6 min    | Operators, Admins  | Videos 1-2       |
| 8  | Quota and Metering           | 8 min    | All users          | Video 1          |

**Total Series Duration**: ~74 minutes

---

## Video 1: Platform Overview

**Duration**: 8 minutes
**Presenter**: Platform Engineering Lead
**Screen Recording**: DBaaS Dashboard

### Script

---

**[INTRO - 0:00-0:30]**

**[Show: Title card with "ERP-DBaaS: Platform Overview" text on blue (#2563eb) background]**

NARRATOR: "Welcome to ERP-DBaaS, your organization's self-service Database-as-a-Service platform. In this video, we will walk you through what DBaaS is, why it was built, and how to navigate the platform. By the end, you will have a solid understanding of the platform's capabilities and be ready to start provisioning databases."

---

**[SECTION 1: What is DBaaS? - 0:30-2:00]**

**[Show: Animated diagram of the old process: Developer -> Jira Ticket -> DBA Queue -> Manual Setup -> Credentials via Email -> Developer (with "2-5 days" label)]**

NARRATOR: "Before DBaaS, getting a new database was a multi-day process. You would file a Jira ticket, wait for a DBA to pick it up, and then wait several more days for manual provisioning and configuration. Credentials were shared over email, and there was no standardized way to manage backups or monitor health."

**[Show: Animated diagram of the new process: Developer -> DBaaS Wizard -> Automated Provisioning -> Instant Credentials -> Developer (with "< 5 minutes" label)]**

NARRATOR: "ERP-DBaaS changes this completely. Through a self-service web interface, you can provision a production-grade database in under 5 minutes. The platform handles security configuration, backup schedules, credential management, and monitoring automatically -- all governed by organizational policies."

---

**[SECTION 2: Supported Engines - 2:00-3:30]**

**[Show: Engine catalog page with 8 engine cards]**

NARRATOR: "DBaaS supports eight database engines, each optimized for different workloads."

**[Highlight each engine card as it is mentioned]**

NARRATOR: "YugabyteDB for distributed SQL -- this is your go-to for most ERP module databases. ScyllaDB for high-throughput event data. DragonflyDB for caching and real-time counters. MongoDB for flexible document storage. CouchDB for offline-first applications with sync. ClickHouse for analytics and reporting. TimescaleDB and QuestDB for time-series data."

NARRATOR: "Do not worry about memorizing all of these now. The provisioning wizard includes recommendations to help you choose the right engine for your workload."

---

**[SECTION 3: Architecture Overview - 3:30-4:30]**

**[Show: Simplified architecture diagram (animated build-up)]**

NARRATOR: "Under the hood, DBaaS runs on Kubernetes. When you create a database, your request flows through the API gateway, which authenticates you and checks rate limits. The API server evaluates your configuration against governance policies. If approved, a Kubernetes operator provisions your database as isolated pods in a dedicated tenant namespace."

**[Highlight: Operator -> Database Pods animation]**

NARRATOR: "Each database engine has its own dedicated operator that understands the specific lifecycle management requirements of that engine -- from initial provisioning to scaling, backup, and eventually decommissioning."

---

**[SECTION 4: Dashboard Tour - 4:30-6:30]**

**[Show: Live screen recording of the DBaaS dashboard]**

NARRATOR: "Let me walk you through the main dashboard. At the top, you can see summary cards showing your total instances, how many are running, and any that might need attention."

**[Mouse hover over summary cards]**

NARRATOR: "Below that, the health distribution chart gives you a quick visual of how your databases are doing. Green means healthy, yellow means there is a warning, and red means something needs immediate attention."

**[Scroll to quota section]**

NARRATOR: "The quota usage bars show how much of your allocated resources you are currently using. If you are approaching limits, you will see warnings here."

**[Click on Instances in sidebar]**

NARRATOR: "The Instances page gives you a full list of all your databases. You can filter by engine, status, or size, and search by name. Click any instance to see its details, metrics, and backup history."

**[Click on an instance to show detail page]**

NARRATOR: "The instance detail page is your command center for a specific database. Here you can see the connection endpoint, current health, resource utilization, and take actions like scaling, backing up, or rotating credentials."

---

**[SECTION 5: Roles and Access - 6:30-7:30]**

**[Show: RBAC roles table graphic]**

NARRATOR: "DBaaS uses three roles for access control. Viewers can see instances and metrics but cannot make changes. Operators can provision, scale, and manage databases within their tenant. Admins have full access including tenant management and policy configuration."

NARRATOR: "Your role is assigned based on your Authentik identity. If you need a different role, contact your team lead or IT administrator."

---

**[OUTRO - 7:30-8:00]**

**[Show: Title card with series overview]**

NARRATOR: "That covers the platform overview. In the next video, we will walk through provisioning your first database step by step. If you have questions, reach out on the dbaas-support Slack channel. Thanks for watching."

**[Show: End card with "Next: Video 2 - Provisioning a Database"]**

---

## Video 2: Provisioning a Database

**Duration**: 12 minutes
**Presenter**: Platform Engineering Lead
**Screen Recording**: DBaaS Provisioning Wizard

### Script

---

**[INTRO - 0:00-0:30]**

**[Show: Title card "Provisioning a Database"]**

NARRATOR: "In this video, we will walk through the complete process of provisioning a new database instance using the DBaaS provisioning wizard. We will cover engine selection, configuration, policy evaluation, and what happens after you click Provision."

---

**[SECTION 1: Starting the Wizard - 0:30-1:30]**

**[Show: Dashboard with cursor moving to "+ New Instance" button]**

NARRATOR: "Start by clicking the 'New Instance' button in the top-right corner of the dashboard. This opens the four-step provisioning wizard."

**[Click button, wizard opens]**

NARRATOR: "The wizard guides you through four steps: Engine Selection, Configuration, Policy Review, and Confirmation. Let us walk through each step."

---

**[SECTION 2: Engine Selection - 1:30-4:00]**

**[Show: Engine selection screen with 8 cards]**

NARRATOR: "Step one is choosing your database engine. You will see eight cards, each representing a supported engine. Let me help you choose."

NARRATOR: "For this demo, we are building a new ERP Procurement module that needs a primary relational database. YugabyteDB is the recommended choice because it is PostgreSQL-compatible, supports distributed transactions, and provides built-in high availability."

**[Click "Learn More" on YugabyteDB card]**

NARRATOR: "Clicking 'Learn More' shows you detailed information about the engine, including supported versions, deployment modes, and recommended use cases."

**[Close detail, select YugabyteDB]**

NARRATOR: "Let us select YugabyteDB and click Next."

**[Click Next]**

---

**[SECTION 3: Configuration - 4:00-7:30]**

**[Show: Configuration form]**

NARRATOR: "Step two is where we configure our instance. Let us start with the name."

**[Type "procurement-primary" in the name field]**

NARRATOR: "I am naming this 'procurement-primary'. Names must be lowercase with hyphens, between 3 and 63 characters. The platform will automatically prepend your tenant name."

**[Select version dropdown]**

NARRATOR: "For the version, I will choose 2.20, which is the latest stable release, marked with a star."

**[Select Medium size preset]**

NARRATOR: "For the size, I will choose Medium -- 2 vCPU, 4 GB RAM, and 50 GB storage. This is appropriate for a new module in early production."

**[Adjust replica slider to 3]**

NARRATOR: "Since this is a production database, I need at least 3 replicas for high availability. The slider shows me the quota impact as I adjust."

**[Expand backup section]**

NARRATOR: "Now for backups. I will select 'Every 6 Hours' for the schedule, and set retention to 90 days, which aligns with our compliance requirements."

**[Review the quota impact preview on the right side]**

NARRATOR: "Notice the quota impact panel on the right. It shows that this instance will use 6 vCPU, 12 GB memory, and 150 GB storage across all 3 replicas. My tenant has sufficient quota available."

**[Click Next]**

---

**[SECTION 4: Policy Review - 7:30-9:30]**

**[Show: Policy review screen with all green checkmarks]**

NARRATOR: "Step three is the AIDD policy review. The platform evaluates our configuration against the active policy profile -- in this case, the strict production profile."

NARRATOR: "Great news -- all checks pass. We see green checkmarks for security requirements, high availability, backup configuration, and resource quotas."

**[Scroll through the rules]**

NARRATOR: "Let me scroll through the checks. Encryption at rest is enabled, TLS is configured, we have 3 replicas meeting the HA requirement, backups are scheduled with sufficient retention, and we are within our quota."

**[Point to recommendations section]**

NARRATOR: "The platform also provides optional recommendations. Here it suggests considering a 5-node cluster for enhanced availability, though 3 nodes is perfectly acceptable."

NARRATOR: "If any checks had failed, we would see red violations with specific guidance on how to fix them. The 'Fix' button can automatically adjust our configuration."

**[Click Next]**

---

**[SECTION 5: Review and Confirm - 9:30-10:30]**

**[Show: Summary page]**

NARRATOR: "Step four is the final review. We see a complete summary of our configuration: YugabyteDB 2.20, Medium size, 3 replicas, backup every 6 hours with 90-day retention."

NARRATOR: "At the bottom, we see the estimated monthly cost for internal chargeback purposes. Everything looks good."

**[Click "Provision"]**

NARRATOR: "I will click Provision to start the process."

---

**[SECTION 6: Provisioning Progress - 10:30-11:30]**

**[Show: Instance detail page with provisioning progress]**

NARRATOR: "We are now on the instance detail page, and you can see the provisioning progress in real-time. The status bar shows each stage."

**[Show stages animating through: Pending -> Creating Resources -> Deploying StatefulSet -> Initializing -> Configuring Backups -> Health Check]**

NARRATOR: "The platform is creating namespace resources, deploying the StatefulSet, initializing the database, configuring backups, and running health checks. This typically takes 2-3 minutes for a medium YugabyteDB instance."

**[Status changes to "Running"]**

NARRATOR: "And we are done. The instance is now running. You can see the connection endpoint, and credentials are available in the Connection Information section."

---

**[OUTRO - 11:30-12:00]**

NARRATOR: "That is the complete provisioning workflow. What used to take days now takes minutes. In the next video, we will cover scaling and high availability. See you there."

**[Show: End card]**

---

## Video 3: Scaling and High Availability

**Duration**: 10 minutes
**Presenter**: Platform Engineering Lead

### Script

---

**[INTRO - 0:00-0:30]**

**[Show: Title card "Scaling and High Availability"]**

NARRATOR: "In this video, we will cover how to scale your database instances to handle increased workload, and how high availability works in DBaaS. We will demonstrate vertical scaling, horizontal scaling, and storage expansion."

---

**[SECTION 1: When to Scale - 0:30-2:30]**

**[Show: Instance metrics dashboard with increasing CPU graph]**

NARRATOR: "Let us start with understanding when you need to scale. The metrics tab on your instance detail page is your best friend here."

NARRATOR: "This instance has been showing CPU utilization consistently above 75 percent for the past week. That is our signal to scale up. Other signs include increasing memory pressure, storage approaching capacity, growing connection counts, or rising query latency."

**[Show: Scaling decision matrix graphic]**

NARRATOR: "Here is a quick reference: if you see high CPU, increase CPU allocation. High memory, increase memory. Storage running out, expand storage. Too many connections, add replicas. It is that straightforward."

---

**[SECTION 2: Vertical Scaling Demo - 2:30-5:00]**

**[Show: Instance detail page, click Scale button]**

NARRATOR: "Let me demonstrate vertical scaling. I will click the Scale button on this instance that is currently on the Medium preset."

**[Show: Scaling dialog]**

NARRATOR: "The scaling dialog shows our current allocation on the left and the new allocation on the right. I am going to upgrade from Medium to Large, which doubles our CPU to 4 cores and memory to 8 GB."

**[Select Large preset]**

NARRATOR: "Notice the quota impact preview updates in real-time. Since we have 3 replicas, this change will consume an additional 6 vCPU and 12 GB of memory across all replicas."

NARRATOR: "For HA instances like this one, the scaling is performed as a rolling update. One pod at a time is restarted with the new resources, so there is zero downtime for your application."

**[Click Apply Scale]**

NARRATOR: "After clicking Apply Scale, you can see the instance status changes to 'Scaling'. The progress indicator shows which pod is currently being updated."

**[Show rolling update progress]**

NARRATOR: "Pod 1 of 3 updated... pod 2 of 3... and pod 3 of 3. The instance is back to Running status with the new resources. The entire process took about 2 minutes."

---

**[SECTION 3: Horizontal Scaling Demo - 5:00-7:00]**

**[Show: Scale dialog, replicas tab]**

NARRATOR: "Now let us look at horizontal scaling. This is useful when you need more capacity for handling concurrent connections or distributing read workload across more nodes."

NARRATOR: "In the Scale dialog, I switch to the Replicas tab. Currently we have 3 replicas. I will increase to 5 for enhanced availability."

**[Adjust slider to 5]**

NARRATOR: "The new pods will automatically join the cluster and receive data through the engine's replication mechanism. For YugabyteDB, this uses Raft consensus to distribute data across the new nodes."

**[Click Apply Scale]**

NARRATOR: "The scaling process creates the new pods, waits for data synchronization, and then marks them as ready. This takes a bit longer than vertical scaling because data needs to be rebalanced."

---

**[SECTION 4: Storage Expansion - 7:00-8:00]**

**[Show: Scale dialog, storage tab]**

NARRATOR: "Storage expansion is the simplest scaling operation. In the Storage tab, enter the new storage size. Remember, storage can only grow -- it cannot be reduced."

**[Change from 50GB to 100GB]**

NARRATOR: "I am expanding from 50 GB to 100 GB. The expansion uses Kubernetes PersistentVolumeClaim resizing, which is typically non-disruptive. The new space is available immediately."

---

**[SECTION 5: High Availability Explained - 8:00-9:30]**

**[Show: HA architecture diagram animation]**

NARRATOR: "Let me explain how high availability works. When you have 3 or more replicas, the database engine distributes data across all nodes using replication."

NARRATOR: "If one node fails, the remaining nodes continue serving traffic. The operator detects the failure and automatically schedules a replacement pod. Data is rebalanced, and the cluster returns to full health -- all without manual intervention."

NARRATOR: "For YugabyteDB with RF3, any single node failure is tolerated without data loss. For RF5, up to two simultaneous node failures can be survived."

---

**[OUTRO - 9:30-10:00]**

NARRATOR: "Scaling in DBaaS is designed to be safe, predictable, and non-disruptive. Always check your metrics before scaling, verify your quota, and monitor the process. In the next video, we will cover backup and restore operations."

**[Show: End card]**

---

## Video 4: Backup and Restore

**Duration**: 12 minutes
**Presenter**: Database Reliability Engineer

### Script

---

**[INTRO - 0:00-0:30]**

**[Show: Title card "Backup and Restore"]**

NARRATOR: "Backups are your safety net. In this video, we will cover how DBaaS manages automated backups, how to trigger on-demand backups, and how to restore from a backup. We will also demonstrate point-in-time recovery for supported engines."

---

**[SECTION 1: Backup Overview - 0:30-2:30]**

**[Show: Backup concepts animation]**

NARRATOR: "DBaaS supports three types of backups. Full backups capture everything -- a complete snapshot of your data. They are self-contained and can be restored independently."

NARRATOR: "Incremental backups capture only the changes since the last backup. They are faster and smaller, but require the base full backup to restore."

NARRATOR: "Point-in-time recovery, or PITR, continuously archives transaction logs, allowing you to restore to any specific second. This is available for YugabyteDB, TimescaleDB, and MongoDB."

---

**[SECTION 2: Configuring Backups - 2:30-5:00]**

**[Show: Instance Backups tab]**

NARRATOR: "Let me show you how to configure a backup schedule. On the instance detail page, click the Backups tab, then Edit Schedule."

**[Click Edit Schedule]**

NARRATOR: "I will set up a production-grade backup configuration. For the schedule, I am choosing 'Every 6 Hours', which creates backups at midnight, 6 AM, noon, and 6 PM UTC."

NARRATOR: "For backup type, I will use Full for the daily backup and Incremental for the intermediate backups. The retention is set to 90 days per our compliance policy."

NARRATOR: "Encryption is enabled by default for all backups using your tenant's encryption key. Backups are stored in RustFS, the platform's S3-compatible object storage."

**[Click Save]**

NARRATOR: "After saving, you can see the next scheduled backup time and the full schedule displayed."

---

**[SECTION 3: On-Demand Backup Demo - 5:00-7:00]**

**[Show: Click "Backup Now" button]**

NARRATOR: "Sometimes you need an immediate backup -- before a migration, a schema change, or any risky operation. Click 'Backup Now' to trigger an on-demand backup."

NARRATOR: "I will select Full backup type and click Start."

**[Show: Backup progress]**

NARRATOR: "The backup progress shows the current phase: initializing, dumping data, uploading to RustFS, and verifying checksum. For this medium-sized database, it takes about 2 minutes."

**[Backup completes]**

NARRATOR: "The backup is complete. In the history, you can see the backup size, duration, checksum, and storage location. The green checkmark confirms the integrity verification passed."

---

**[SECTION 4: Restore Demo - 7:00-10:30]**

**[Show: Click Restore button on a backup]**

NARRATOR: "Now for the restore. I will click the Restore button next to this backup. This opens the restore dialog."

NARRATOR: "I always recommend restoring to a new instance rather than overwriting the original. This way, you can validate the restored data before switching your application."

**[Select "New Instance", enter name]**

NARRATOR: "I will name the restored instance 'procurement-primary-restored' and keep the same size preset."

**[Click Start Restore]**

NARRATOR: "The restore process begins. It downloads the backup from RustFS, verifies the checksum, creates a new instance, and loads the data."

**[Show: Restore progress animation]**

NARRATOR: "The progress bar shows the restoration percentage. For this backup, we are at 25 percent... 50 percent... 75 percent... and 100 percent."

**[Instance reaches Running status]**

NARRATOR: "The restored instance is now running. Notice it has new credentials -- the original credentials are not reused for security reasons. Update your application's connection settings to use the new endpoint."

---

**[SECTION 5: PITR Demo - 10:30-11:30]**

**[Show: PITR restore option]**

NARRATOR: "For engines that support point-in-time recovery, you will see a PITR option in the restore dialog. Instead of selecting a specific backup, you choose a timestamp."

NARRATOR: "Select the date and time you want to restore to. The platform will find the closest full backup and replay transaction logs up to your specified point. This is invaluable for recovering from accidental data deletions or corruption."

---

**[OUTRO - 11:30-12:00]**

NARRATOR: "Remember: backups are only as good as your ability to restore from them. Practice your restore workflow periodically. In the next video, we dive into the engine catalog for a detailed comparison of all eight supported engines."

**[Show: End card]**

---

## Video 5: Engine Catalog Deep Dive

**Duration**: 10 minutes
**Presenter**: Senior DBA

### Script

---

**[INTRO - 0:00-0:30]**

**[Show: Title card "Engine Catalog Deep Dive"]**

NARRATOR: "Choosing the right database engine is one of the most important decisions for your application. In this video, we will explore each of the eight supported engines, when to use them, and how to compare them using the engine catalog."

---

**[SECTION 1: Navigating the Catalog - 0:30-2:00]**

**[Show: Engine Catalog page]**

NARRATOR: "The Engine Catalog is accessible from the left sidebar. You will see all eight engines displayed as detailed cards with key attributes."

NARRATOR: "Each card shows the engine name, category, a brief description, and key metrics like maximum throughput and latency characteristics. You can filter engines by category: Relational, Document, Key-Value, Columnar, Time-Series, and Wide-Column."

**[Click on comparison view]**

NARRATOR: "The comparison view lets you select two or three engines side by side. This is especially useful when you are deciding between similar options."

---

**[SECTION 2: Engine Walkthroughs - 2:00-8:30]**

**[Show each engine detail page for approximately 45 seconds each]**

NARRATOR: "Let us walk through each engine."

**[YugabyteDB detail page]**
NARRATOR: "YugabyteDB is your default choice for relational workloads. It speaks PostgreSQL, supports distributed ACID transactions, and scales horizontally. If your ERP module needs a primary database with complex queries and joins, start here."

**[ScyllaDB detail page]**
NARRATOR: "ScyllaDB excels at extreme throughput with consistent low latency. Think audit logs with millions of writes per second, or event stores where you always query by a known partition key."

**[DragonflyDB detail page]**
NARRATOR: "DragonflyDB is your in-memory data store. Redis-compatible but faster and more memory-efficient. Perfect for caching, session management, and rate limiting."

**[MongoDB detail page]**
NARRATOR: "MongoDB is ideal when your data schema is evolving rapidly or does not fit neatly into tables. Product catalogs, user preferences, and content management are classic use cases."

**[CouchDB detail page]**
NARRATOR: "CouchDB is unique for its multi-master replication. If you need offline-first capabilities or applications that sync data from multiple locations, CouchDB is your answer."

**[ClickHouse detail page]**
NARRATOR: "ClickHouse is an analytical powerhouse. It can scan billions of rows per second for aggregate queries. Use it for dashboards, reports, and data warehousing. It is not for transactional workloads."

**[TimescaleDB detail page]**
NARRATOR: "TimescaleDB extends PostgreSQL with time-series superpowers. Automatic partitioning, continuous aggregates, and compression. Great for IoT data, application metrics, and any time-stamped relational data."

**[QuestDB detail page]**
NARRATOR: "QuestDB is built for maximum time-series ingest speed. If you have financial tick data or sensor streams with millions of data points per second, QuestDB handles it with minimal resources."

---

**[SECTION 3: Choosing the Right Engine - 8:30-9:30]**

**[Show: Decision flowchart animation]**

NARRATOR: "Here is a simple decision framework. If you need SQL with transactions, go with YugabyteDB. If you need analytics, ClickHouse. If you need caching, DragonflyDB. For time-series, TimescaleDB or QuestDB depending on your ingest volume. For documents, MongoDB or CouchDB based on whether you need sync. And for extreme write throughput with predictable queries, ScyllaDB."

---

**[OUTRO - 9:30-10:00]**

NARRATOR: "Still not sure which engine to choose? The provisioning wizard includes a recommendation engine that asks about your workload and suggests the best fit. Or reach out to the DBA team on Slack for personalized guidance."

**[Show: End card]**

---

## Video 6: Plugin Management

**Duration**: 8 minutes
**Presenter**: Platform Engineer

### Script

---

**[INTRO - 0:00-0:20]**

**[Show: Title card "Plugin Management"]**

NARRATOR: "DBaaS is extensible through its plugin system. In this video, we will cover how plugins work, how to register a new plugin, and how to manage plugin health. This video is aimed at platform administrators."

---

**[SECTION 1: Plugin Concepts - 0:20-2:00]**

**[Show: Plugin architecture diagram]**

NARRATOR: "Plugins are containerized gRPC services that hook into the DBaaS lifecycle. When events like provisioning, backup, or credential rotation occur, the platform notifies all plugins subscribed to those hooks."

NARRATOR: "The ten available hook points are: preProvision, postProvision, preScale, postScale, preBackup, postBackup, preRestore, postRestore, onCredentialRotation, and onDecommission."

NARRATOR: "Pre-hooks can block an operation. For example, a validation plugin on preProvision could reject an instance that does not meet custom naming conventions. Post-hooks are informational and cannot block the operation."

---

**[SECTION 2: Registering a Plugin - 2:00-5:00]**

**[Show: Plugin registration form]**

NARRATOR: "Let me register a new plugin. I will navigate to Plugins and click Register Plugin."

**[Fill in form fields]**

NARRATOR: "I am registering a Slack notification plugin. The name is 'slack-lifecycle-notifier', version 1.0.0, type is Notification. The container image is in our internal registry."

NARRATOR: "For hooks, I am subscribing to postProvision, postBackup, and onCredentialRotation. This means the plugin will be notified after every new instance is provisioned, every backup completes, and every credential rotation."

NARRATOR: "I set resource limits to 100 milliCPU and 128 megabytes of memory. Plugins should be lightweight."

**[Click Register]**

NARRATOR: "After clicking Register, the platform pulls the image, starts the container, and runs health checks. We can see the status progressing from Pulling to Starting to Active."

---

**[SECTION 3: Monitoring Plugins - 5:00-7:00]**

**[Show: Plugin status dashboard]**

NARRATOR: "The plugin dashboard shows all registered plugins with their current status. Green means healthy and processing hooks. Yellow means the plugin has had recent errors but is still running. Red means the plugin is unresponsive."

NARRATOR: "Click on any plugin to see detailed information: recent hook executions, response times, error logs, and resource utilization."

NARRATOR: "If a plugin becomes unhealthy, the platform will automatically restart it. After three consecutive restart failures, the plugin is marked as failed and hook delivery is suspended."

---

**[SECTION 4: Managing Plugin Lifecycle - 7:00-7:40]**

NARRATOR: "From the plugin detail page, you can disable a plugin to temporarily stop hook delivery, update its container image to deploy a new version, or remove it entirely."

NARRATOR: "When updating a plugin, the new version is started first, health-checked, and then the old version is terminated. This ensures zero gap in hook processing."

---

**[OUTRO - 7:40-8:00]**

NARRATOR: "Plugins make DBaaS infinitely extensible. Whether you need custom notifications, validation rules, or integration with external systems, the plugin framework has you covered."

**[Show: End card]**

---

## Video 7: Credential Rotation

**Duration**: 6 minutes
**Presenter**: Security Engineer

### Script

---

**[INTRO - 0:00-0:20]**

**[Show: Title card "Credential Rotation"]**

NARRATOR: "Credential rotation is a critical security practice. In this video, we will demonstrate how to rotate database credentials, understand the grace period mechanism, and configure automatic rotation schedules."

---

**[SECTION 1: Why Rotate? - 0:20-1:30]**

NARRATOR: "Credentials should be rotated regularly to limit the impact of any potential credential leak. Our AIDD governance policy recommends rotation at least every 90 days. Some compliance frameworks require even more frequent rotation."

NARRATOR: "DBaaS makes rotation seamless with a built-in grace period that allows your application to transition to new credentials without downtime."

---

**[SECTION 2: Manual Rotation Demo - 1:30-3:30]**

**[Show: Instance detail page, Connection Information section]**

NARRATOR: "Let me walk through a manual rotation. On the instance detail page, I will click 'Rotate Credentials'."

**[Click Rotate Credentials]**

NARRATOR: "A confirmation dialog explains what will happen. The new credentials will be generated immediately, and the old credentials will remain valid for 5 minutes."

**[Click Confirm]**

NARRATOR: "New credentials are generated. Notice the Connection Information section now shows updated values. The password is displayed once -- make sure to copy it now or use the download button."

NARRATOR: "You have 5 minutes to update your application configuration. The old credentials continue to work during this grace period, so there is no disruption."

**[Show: Rotation history]**

NARRATOR: "Every rotation is logged in the Credential Rotation History with the timestamp, who initiated it, and whether it was manual or scheduled."

---

**[SECTION 3: Automatic Rotation - 3:30-5:00]**

**[Show: Configuration tab, Auto-rotation settings]**

NARRATOR: "For production instances, I recommend configuring automatic rotation. On the Configuration tab, find Auto-Rotation Settings."

NARRATOR: "Set the rotation interval -- for example, every 60 days. The platform will automatically rotate credentials on schedule and send notifications 7 days before rotation and immediately after."

NARRATOR: "If your application reads credentials from Kubernetes Secrets, the Secret is updated automatically and your application will pick up the new credentials on its next connection."

---

**[SECTION 4: Best Practices - 5:00-5:40]**

NARRATOR: "Security best practices for DBaaS credentials. First, never hard-code credentials in your application code. Use environment variables or Kubernetes Secret mounts. Second, enable automatic rotation for all production instances. Third, monitor the rotation history for any unexpected rotations. And finally, test your application's ability to handle credential changes gracefully."

---

**[OUTRO - 5:40-6:00]**

NARRATOR: "Credential rotation in DBaaS is designed to be zero-downtime and fully automated. Set it up once and let the platform handle the rest."

**[Show: End card]**

---

## Video 8: Quota and Metering

**Duration**: 8 minutes
**Presenter**: FinOps Engineer

### Script

---

**[INTRO - 0:00-0:20]**

**[Show: Title card "Quota and Metering"]**

NARRATOR: "Understanding your resource consumption and costs is essential for efficient operations. In this video, we will cover quota management, metering dashboards, cost attribution, and how to request quota increases."

---

**[SECTION 1: Understanding Quotas - 0:20-2:00]**

**[Show: Quota dashboard]**

NARRATOR: "Every tenant in DBaaS has resource quotas that limit how much infrastructure they can consume. This ensures fair resource allocation and prevents any single tenant from exhausting shared cluster resources."

**[Show tier comparison table]**

NARRATOR: "Quotas are tier-based. Free tier tenants get 5 instances, 8 vCPU, and 16 GB of memory. Standard tier gets 25 instances, 64 vCPU, and 128 GB. Enterprise tier quotas are customized per organization."

NARRATOR: "The quota dashboard shows your current usage as progress bars with color coding. Green under 70 percent, yellow from 70 to 90 percent, and red above 90 percent."

---

**[SECTION 2: Metering Dashboard - 2:00-4:30]**

**[Show: Metering page with graphs]**

NARRATOR: "The Metering dashboard provides detailed resource consumption data for cost attribution."

NARRATOR: "The Usage Timeline shows your resource consumption over time. You can see CPU-hours, memory gigabyte-hours, and storage gigabyte-months. Toggle between daily, weekly, and monthly views."

**[Show: Cost breakdown pie chart]**

NARRATOR: "The Cost Breakdown view shows your estimated costs by engine and by instance. This helps you identify your most expensive databases and optimize resource allocation."

**[Show: Top consumers table]**

NARRATOR: "The Top Consumers table ranks your instances by estimated monthly cost. If you are looking to reduce spend, start with the top entries and see if they can be right-sized."

---

**[SECTION 3: Requesting Quota Increases - 4:30-6:00]**

**[Show: Click "Request Increase"]**

NARRATOR: "If you are approaching your quota limits and need more resources, click 'Request Increase' on the Quotas page."

**[Show: Request form]**

NARRATOR: "Select which resources you need increased, enter the requested amounts, and provide a business justification. For example, 'Launching 3 new microservices for the inventory module rewrite, requiring 3 additional medium-sized YugabyteDB instances.'"

NARRATOR: "The request goes to the platform admin team for review. Standard requests are typically approved within one business day. Enterprise tier requests may require FinOps approval."

---

**[SECTION 4: Cost Optimization Tips - 6:00-7:30]**

NARRATOR: "Let me share some cost optimization tips."

NARRATOR: "First, right-size your instances. If an instance consistently uses less than 30 percent of its CPU, consider downsizing to a smaller preset."

NARRATOR: "Second, decommission unused instances. Development databases that have not been accessed in over 30 days are flagged on the dashboard."

NARRATOR: "Third, use DragonflyDB for caching instead of over-provisioning your primary database. Offloading read traffic to a cache is often more cost-effective than scaling up."

NARRATOR: "Fourth, review backup retention. Keeping 365 days of daily backups may not be necessary for non-production environments."

---

**[OUTRO - 7:30-8:00]**

NARRATOR: "Visibility into costs and quotas enables data-driven decisions about your database infrastructure. Use the metering dashboard regularly to stay on top of your resource consumption."

**[Show: End card with series summary]**

NARRATOR: "That concludes the ERP-DBaaS video training series. Remember, the dbaas-support Slack channel is always available for questions. Happy provisioning."

---

## Production Notes

### Recording Requirements
- Screen resolution: 1920x1080
- Browser: Chrome (latest stable)
- Font size: Increased to 125% for readability
- Mouse movements: Slow and deliberate for clarity
- Annotations: Use Keynote or Figma for diagrams
- Audio: Professional narration with background music (low volume)

### Editing Guidelines
- Add chapter markers at each section boundary
- Include on-screen text for key terms (first mention)
- Add zoom effects when showing small UI elements
- Use transition effects between sections (subtle fade)
- Include captions/subtitles for accessibility
- Add end card with next video link and support channel

### Distribution
- Host on internal LMS (Learning Management System)
- Also available on SharePoint video library
- Thumbnail format: 1280x720 with title text and blue (#2563eb) branding

---

*These scripts are maintained by the Training and Enablement team. Last updated: February 24, 2026.*
