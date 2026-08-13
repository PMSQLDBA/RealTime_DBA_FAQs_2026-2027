It is an **Application SQL Server DBA interview** focused heavily on SQL Server administration, migration, replication, performance troubleshooting, archiving, Azure migration, Standard-vs-Enterprise limitations, bidirectional synchronization, and regional data security. The role itself includes production changes, permissions, maintenance, slow-job troubleshooting, and working alongside the development and enterprise DBA teams. 

Below, I’ve reconstructed the interview questions and given stronger answers aligned with current Microsoft guidance.

## 1. Tell me about yourself / your DBA background.

The transcript describes roughly 14–15 years of SQL DBA experience, SQL Server migrations, application/on-prem DBA work, ServiceNow/CAB change management, and coordination with application, development, infrastructure, and database teams. 

**Strong interview answer:**

“I have around 14+ years of SQL Server DBA experience across production support, application DBA activities, migrations, performance tuning, high availability, security, change management, and database maintenance.

I’ve worked with application and development teams as well as infrastructure and enterprise DBA teams. My migration experience includes SQL Server version upgrades and moving databases between environments, including Azure-related workloads.

From an application DBA perspective, I’m comfortable with deployments through controlled change processes, database roles and permissions, SQL Agent jobs, troubleshooting performance problems, reviewing execution plans, indexes and statistics, and coordinating production releases through CAB/change-management processes.

For production work, my approach is always to validate in non-production first, document the implementation and rollback plans, execute through an approved change window, validate afterward, and monitor the environment.”

---

# SQL Server Migration

## 2. What types of SQL Server migrations have you performed?

This was one of the first major technical questions in the interview. 

**Strong answer:**

“I classify SQL migrations into three broad categories: version upgrades, infrastructure/platform migrations, and cloud migrations.

Before migration I inventory databases, SQL Agent jobs, logins/SIDs, linked servers, credentials, certificates, SSIS dependencies and application connections. I verify SQL Server version/edition compatibility, database compatibility levels, deprecated features, HA/DR dependencies, storage requirements, and application dependencies.

Then I establish performance and workload baselines and select the migration mechanism based on database size and allowable downtime—for example backup/restore, log backup/restore sequencing, availability technologies, or Azure migration services.

Before cutover I perform test migrations and application validation. During cutover I stop or control application writes, perform final synchronization, validate database consistency and data, redirect application connectivity, perform smoke tests, and monitor performance.

Most importantly, I always maintain a documented rollback plan.”

Microsoft's current migration guidance similarly emphasizes assessment, migration method selection, validation testing, performance comparison, synchronization, and coordinated cutover. ([Microsoft Learn][1])

---

## 3. What is the largest database you have managed?

The transcript answer was approximately **2 TB**. 

A better interview response would add operational context:

“I've managed databases around 2 TB. For databases at that scale, I pay particular attention to backup/restore duration, RPO/RTO, transaction-log growth, storage latency and throughput, index maintenance strategy, statistics, integrity checks, HA/DR, capacity planning, and the time required for migrations or recovery.

I wouldn't evaluate a 2-TB database only by its size—the workload, change rate, I/O characteristics, recovery objectives and maintenance windows are equally important.”

---

# Replication

## 4. What SQL Server replication types do you know?

The original answer only mentioned snapshot and transactional replication. 

**Better answer:**

“The major SQL Server replication models I would discuss are:

* **Snapshot replication** — distributes a point-in-time snapshot; appropriate when data changes infrequently or periodic complete refreshes are acceptable.
* **Transactional replication** — takes an initial snapshot and then continuously propagates committed changes from the Publisher through the Distributor to Subscribers. It's commonly used when low-latency one-directional synchronization is required.
* **Merge replication** — allows changes at multiple nodes and later synchronization, including conflict detection/resolution.

I select the technology based on whether synchronization is one-way or multi-writer, acceptable latency, connectivity, conflict requirements, topology, database size, and operational complexity.”

---

## 5. Is snapshot replication bidirectional?

**Answer:**

“No. Snapshot replication is fundamentally publisher-to-subscriber distribution. It sends a point-in-time copy of the published schema and data to Subscribers.

It should not be presented as a bidirectional synchronization technology.”

Microsoft describes snapshot generation as copying the published schema and data into snapshot files that are subsequently distributed to subscribers. ([Microsoft Learn][2])

---

## 6. Can transactional replication replicate only selected tables?

This was specifically asked in the transcript. 

**Answer:**

“Yes. Replication works with **articles**, so I can publish only the required tables instead of the entire database.

For standard transactional replication, every table published as an article must have a primary key.

I can also use row and column filtering where appropriate so that only the necessary data is distributed. Publishing only required data is also a Microsoft replication performance best practice.”

Microsoft confirms that transactional replication requires a primary key on published tables. ([Microsoft Learn][3]) Microsoft also recommends publishing only required data to reduce distribution overhead. ([Microsoft Learn][4])

---

## 7. Does snapshot replication require a primary key?

This is an important correction to the discussion.

**Answer:**

“No. **Snapshot replication does not require a primary key.**

Transactional replication requires a primary key for published tables. Merge replication does not require one, although existing primary keys have replication considerations.”

Microsoft explicitly documents this distinction. ([Microsoft Learn][5])

---

# 25 Years of Data / Archiving

## 8. We have 25 years of data and want only five years active. How would you design the archiving project?

This was one of the most important scenario questions. 

The interview answer jumped almost immediately to filegroups/partitioning. A stronger answer starts with requirements.

**Strong answer:**

“I would not begin by immediately deleting or partitioning data. First I would establish the business and technical requirements.

I would determine:

1. Which tables contain historical data.
2. The retention requirement for each data domain.
3. Relationships and foreign-key dependencies.
4. Whether archived data must remain queryable.
5. Compliance/legal retention requirements.
6. Current table sizes and annual growth.
7. Application queries that reference historical records.
8. Required archive retrieval performance.
9. Backup, recovery and DR requirements for the archive.

Then I would design the archive architecture.

If large tables have reliable date keys, partitioning can simplify lifecycle management. Alternatively, depending on application requirements, I can maintain a separate archive database and move qualifying rows in controlled batches.

I would archive parent/child data carefully so referential integrity remains valid. Moves should be batched to control transaction-log growth, locking and blocking.

After moving data, I validate row counts/checksums and application behavior before deleting anything from the active database.

Finally, I automate the retention process—for example monthly—while monitoring runtime, blocking, transaction-log usage and archive growth.

The archive also needs its own backup, security and recovery strategy.”

That is much stronger than saying simply “create filegroups and partitions.”

---

## 9. Is table partitioning available in SQL Server Standard Edition?

The candidate was uncertain about this. 

**Interview answer:**

“Yes, but I would verify the exact SQL Server version/build before proposing the design because edition capabilities have changed over SQL Server releases.

I wouldn't design an archival strategy based solely on an assumption about Enterprise-only functionality. I would first confirm the production SQL Server version and edition and then validate the feature support.”

This is a useful interview habit: **never guess an edition/version limitation.**

---

# Performance Troubleshooting

## 10. The application becomes slow after moving data. How do you troubleshoot it?

The interview asked for a sequence of steps. 

The candidate mentioned `sp_who`, `sp_WhoIsActive`, execution plans, scans, lookups, missing indexes and non-production testing. 

A stronger Microsoft-aligned answer:

“I first establish scope rather than immediately creating an index.

**1. Establish the symptom.**
Is the entire application slow or only one function? When did it begin? What changed? Is it intermittent or constant?

**2. Check SQL Server and infrastructure health.**
CPU, memory pressure, storage latency, I/O throughput, connection pressure, blocking, waits, transaction-log pressure and errors.

**3. Identify expensive or regressed queries.**
I prefer **Query Store**, DMVs and Extended Events. Query Store is particularly useful for comparing execution plans and runtime statistics before and after a migration or deployment.

**4. Check blocking and waits.**
Determine whether users are waiting on locks, CPU, I/O, memory, network or another resource.

**5. Examine the actual execution plan.**
Look for poor cardinality estimates, inefficient joins, excessive scans, spills, lookups, implicit conversions, parameter-sensitive behavior, and other plan issues.

**6. Check statistics and indexes.**
Confirm statistics are representative and investigate index usage. Missing-index suggestions are input—not something I blindly implement.

**7. Compare application and SSMS behavior.**
Check parameters, SET options and connection characteristics if a query behaves differently through the application.

**8. Validate the fix outside production.**

**9. Deploy through the approved change process with rollback.**

**10. Compare performance after the change against the baseline.**”

Microsoft recommends Query Store as a core query-performance troubleshooting mechanism. ([Microsoft Learn][6]) Microsoft also recommends Extended Events rather than SQL Profiler/SQL Trace for new tracing work because Profiler/Trace are deprecated. ([Microsoft Learn][7])

### Important interview correction

Avoid saying:

> “I'll start Profiler and find the slow query.”

Prefer:

> “I'll start with Query Store/DMVs and use Extended Events when I need targeted capture.”

---

# Azure Migration

## 11. How would you migrate an on-prem SQL Server database to Azure?

The interview discussed Azure SQL Database and SQL Server on Azure VMs. 

**Strong answer:**

“My first decision is not the migration tool—it's the **target architecture**.

I would assess whether the workload belongs on:

* Azure SQL Database,
* Azure SQL Managed Instance, or
* SQL Server on Azure VM.

The decision depends on SQL Server feature dependencies, instance-level functionality, required OS control, application changes, HA/DR requirements, database size, performance requirements and operational model.

Then I perform a migration readiness assessment to identify unsupported features and compatibility issues.

For SQL Managed Instance, Microsoft currently supports options such as Azure Database Migration Service, Managed Instance link, Log Replay Service and native backup/restore depending on downtime and architecture requirements.

For SQL Server on Azure VM, backup/restore is a common migration mechanism, while other approaches can be selected when lower downtime is required.

I baseline the source, test the migration, validate source versus target, plan the cutover and rollback, migrate server-level dependencies where required, and monitor application behavior after cutover.”

Microsoft's current guidance explicitly recommends choosing the target and migration method according to workload, database size, downtime and compatibility requirements. ([Microsoft Learn][1])

---

## 12. What challenges can occur when migrating even a small 5-GB database to Azure?

The original answer concentrated mostly on network bandwidth. 

**Better answer:**

“Database size alone doesn't determine migration complexity. A 5-GB database can still be difficult.

I would check:

* unsupported SQL Server features,
* compatibility and T-SQL differences,
* logins/users and SIDs,
* SQL Agent jobs,
* linked servers,
* CLR or external dependencies,
* application connection strings,
* network latency/firewall/DNS,
* encryption/certificates,
* performance differences,
* collation,
* database configuration,
* cutover downtime and rollback.

So I would assess compatibility first rather than assuming a small database automatically means an easy migration.”

Microsoft specifically calls out unsupported features, networking, login/user migration, Agent jobs, TDE and capacity as common SQL Managed Instance migration considerations. ([Microsoft Learn][1])

---

# Hybrid On-Prem + Cloud Scenario

## 13. We have 25 years on-prem but want only the latest five years in Azure. What would you do?

This scenario appears directly in the transcript. 

**Strong answer:**

“First I would clarify whether Azure is becoming the primary operational system, a reporting/read-only system, or an archive. That changes the design completely.

If only a subset of rows/tables must be copied, I would evaluate **transactional replication with appropriate filtering** or a purpose-built data integration/ETL approach.

If we're actually migrating an entire database to Azure SQL Managed Instance with minimum downtime, I would evaluate Microsoft's migration technologies such as Managed Instance link, DMS or LRS rather than building a custom replication architecture unnecessarily.

For a five-year rolling window, I would also separate two requirements:

1. Initial load of the existing five years.
2. Incremental synchronization of new changes afterward.

Then I would design retention separately. Replication and retention are different problems.

Before production, I would test synchronization latency, data validation, network throughput, failure/recovery behavior, schema changes, monitoring, and cutover.”

Microsoft currently identifies transactional replication as one option for partial migrations of selected tables, while MI Link/LRS/DMS are available for broader migration scenarios. ([Microsoft Learn][8])

---

## 14. Can we move delta data every day/week?

**Answer:**

“Yes. But I would first determine whether the requirement is **scheduled batch movement** or **continuous synchronization**.

For scheduled batch movement, ETL/data-integration processes can identify and transfer deltas.

For near-real-time change propagation, transactional replication may be appropriate for supported topologies.

For migration scenarios involving Azure SQL Managed Instance, Microsoft provides migration technologies that continuously synchronize source changes until cutover.

The correct architecture depends on acceptable latency, volume of change, conflict requirements and whether Azure is temporary migration infrastructure or a permanent second copy.”

---

## 15. How expensive is daily synchronization?

**Answer:**

“I wouldn't estimate it from total database size. I would measure the **change rate**.

The main factors are:

* transaction volume,
* generated transaction-log volume,
* network bandwidth and latency,
* number and width of replicated articles,
* Distributor capacity,
* target write throughput,
* indexes and constraints,
* synchronization frequency,
* Azure compute/storage/network costs,
* retention and monitoring requirements.

For example, a 2-TB database changing only 2 GB/day could be easier to synchronize than a 200-GB database changing 100 GB/day.

So before designing it I would measure daily log generation and peak transaction throughput.”

This answer shows architectural thinking rather than assuming “large database = slow replication.”

---

# Always On / Standard Edition

## 16. Can SQL Server Standard Edition use Always On Availability Groups?

This section of the interview contained an important mistake. 

**Correct answer:**

“SQL Server Standard Edition supports **Basic Availability Groups**, not the full Enterprise availability-group feature set.

A Basic Availability Group is intentionally limited compared with Enterprise—for example, one database per availability group and two replicas.

Most importantly, the secondary replica in a Standard Edition Basic Availability Group is **not a readable reporting replica**.”

Microsoft documentation describes Standard Edition AGs as two replicas—one primary and one secondary used for availability, not readable workloads. ([Microsoft Learn][9])

### Therefore this interview statement was wrong:

> “Standard works and the secondary can be used for read operations.”

For Basic AG on Standard Edition, **do not claim the secondary is available for normal readable workload offloading.**

---

## 17. Is there a SQL Server “Professional Edition”?

This came up explicitly. 

A strong response:

“I don't recognize ‘Professional’ as a normal SQL Server Database Engine edition name. Before making an architecture decision I would confirm it directly from the server rather than argue over terminology.”

Then mention:

```sql
SELECT
    SERVERPROPERTY('Edition')        AS Edition,
    SERVERPROPERTY('ProductVersion') AS ProductVersion,
    SERVERPROPERTY('ProductLevel')   AS ProductLevel;
```

That is an excellent DBA interview response because you **verify the environment**.

---

# Bidirectional Data

## 18. Cloud users update 10 tables and those changes must return to on-prem. How would you synchronize them?

The transcript proposed transactional replication with updating subscribers. 

This answer needs a significant correction.

**Strong answer:**

“I would first clarify whether both locations are allowed to update the **same rows**.

If only one location owns each row, I would prefer an architecture that partitions write ownership because it substantially reduces conflict complexity.

I would not design a new solution around legacy **transactional replication updatable subscriptions**. Microsoft specifically says that feature should be avoided for new development and planned out of existing applications.

Depending on the requirements, I would evaluate supported replication/data-integration patterns and explicitly design conflict handling.

If multiple locations can update the same logical record, we need clearly defined ownership and conflict resolution—not just replication.”

Microsoft explicitly says transactional replication **updatable subscriptions should be avoided for new development**. ([Microsoft Learn][10])

Microsoft also recommends partitioning inserts/updates whenever possible to minimize conflicts. ([Microsoft Learn][11])

This is one of the biggest corrections from the original interview.

---

## 19. Is transactional replication automatically two-way?

**Answer:**

“No.

Normal transactional replication should be thought of as **Publisher → Subscriber** change propagation.

You don't automatically get bidirectional synchronization merely by creating a transactional publication and subscription.

Special topologies exist for multi-node/update scenarios, but they come with significant requirements and limitations. The legacy updatable-subscription feature should not be selected for new development.”

---

## 20. Is replication scheduled or real-time?

The interviewer asked whether it could run daily at 5 AM versus instantaneously. 

**Answer:**

“It depends on the replication type and agent configuration.

Snapshot generation can be scheduled because it creates a point-in-time snapshot.

Transactional replication is normally used for continuous, low-latency change propagation. The Log Reader Agent reads committed transactions from the publication database transaction log and the Distribution Agent delivers replicated commands to Subscribers.

Agents can be configured to run continuously or according to a schedule depending on the topology and operational requirement.

So I would not describe transactional replication simply as ‘run it at 5 AM every day.’ Continuous mode is common when near-real-time synchronization is required.”

---

## 21. How does transactional replication know what changed?

**Answer:**

“It does not normally scan every table looking for changed rows.

For transactional replication, the Log Reader Agent reads replicated changes from the Publisher's **transaction log**, places commands into the distribution database, and the Distribution Agent applies those changes to the Subscriber.

That transaction-log-based mechanism is why transactional replication can provide low-latency incremental propagation.”

Microsoft notes that transactional replication propagates INSERT/UPDATE/DELETE operations to subscribers and normally applies them through generated stored procedures. ([Microsoft Learn][12])

---

# Regional Security / US vs Europe

## 22. US managers must see only US customers and European managers only European customers. Same table/database. How would you design it?

This scenario appears near the end of the interview. 

The candidate proposed Dynamic Data Masking. That is **not the correct primary solution**.

**Correct answer: Row-Level Security.**

“I would first ensure that every row has an authoritative region/tenant/security attribute such as `RegionId`.

Then I would use **Row-Level Security (RLS)** to restrict which rows each security context can access.

For example:

* European role → rows where `RegionId = 'EU'`
* North American role → rows where `RegionId = 'NA'`

The application should authenticate users through a controlled identity/role model and the security predicate should enforce the restriction at the database layer.

If specific columns such as phone numbers or personal identifiers also need to be obscured, Dynamic Data Masking could complement the design—but DDM is not a substitute for row-level authorization.

I would also apply least privilege, auditing, encryption, and appropriate application authorization.”

Microsoft explicitly says Dynamic Data Masking limits exposure of **sensitive fields** and should be combined with other security features such as row-level security. ([Microsoft Learn][13])

### Interview memory rule

**RLS = which rows you may see.**
**DDM = how sensitive column values appear.**

That distinction is very likely to score points in a SQL Server/Azure interview.

---

# Application DBA / Change Management

## 23. How do you deploy database changes into production?

The role specifically expects change-request creation and production deployments. 

**Strong answer:**

“I follow a controlled deployment process.

First, the database change is peer-reviewed and tested in development/non-production. I validate dependencies, locking/performance impact, permissions, deployment duration and rollback strategy.

The change request contains:

* business reason,
* implementation steps,
* affected databases/objects,
* risk and impact,
* validation plan,
* rollback/backout plan,
* expected duration,
* maintenance window,
* owners and approvals.

After CAB approval, I verify backups/recovery readiness, execute during the approved window, capture deployment evidence, perform smoke tests and technical validation, monitor SQL Server/application health, and close the change only after successful validation.

For failed validation, I execute the predefined rollback plan rather than improvising in production.”

---

## 24. How do you handle database roles and permissions?

**Strong answer:**

“I follow least privilege.

I prefer granting permissions to database roles rather than directly to individual users. Users or service accounts become members of the appropriate role.

I separate read, write, execute and administrative permissions. Applications should not normally run with `db_owner` or `sysadmin`.

For service accounts I grant only the permissions required by the application.

All privileged changes should be auditable and processed through the organization's security/change-management process.”

---

# Leadership Questions

## 25. What size DBA team have you worked with?

The transcript describes a team of roughly 35–40+ database professionals. 

A concise answer:

“I've worked in DBA organizations of approximately 35–40 people across L1/L2/L3 and senior/onsite responsibilities. That taught me structured escalation, documentation, handoffs and clear ownership during incidents and changes.”

---

## 26. Have you managed a DBA team?

The transcript describes leading approximately ten people. 

**Answer:**

“Yes. I've led a DBA team of around ten people across different support levels. My responsibilities included workload allocation, technical guidance, escalations, production-change reviews, mentoring junior DBAs and coordinating incidents.

For critical incidents, I focus on assigning clear owners, restoring service first, communicating status, and then conducting root-cause analysis and documenting preventive actions.”

---

# 27. What does an Application DBA do?

This question is implicit throughout the role description.

**Strong answer:**

“I see an Application DBA as the bridge between development/application teams and the enterprise database/platform team.

My responsibilities would include database deployments, schema changes, permissions, SQL Agent jobs, application performance troubleshooting, indexes and statistics, production-change management, release support, database migrations, troubleshooting connectivity and query issues, and escalating infrastructure/platform problems to the enterprise DBA team when appropriate.

The key difference is that I understand both the database platform and how the application actually uses it.”

That matches the responsibilities the interviewer described at the beginning. 

---

# 28. How do you troubleshoot a slow SQL Agent job?

This is especially relevant because the role specifically says the Application DBA is responsible when jobs run slowly. 

**Strong answer:**

“I first compare the current runtime with the job's historical baseline.

Then I identify the slow step and determine whether it's T-SQL, SSIS, PowerShell, backup/restore or another process.

For a SQL step, I investigate:

* execution plan,
* Query Store history,
* blocking,
* waits,
* CPU and I/O,
* statistics,
* index usage,
* parameter behavior,
* transaction-log growth,
* tempdb,
* concurrent workload,
* data-volume growth.

I also determine what changed around the time the runtime increased—deployment, index changes, statistics, configuration, server load or data growth.

I fix the root cause, test it, deploy through change control and monitor subsequent executions.”

---

# Key corrections you should memorize

The transcript contains several areas where I would **change the answer before using it in another interview**:

| Interview topic             | Don't say                                 | Better Microsoft-aligned answer                                                                                   |
| --------------------------- | ----------------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| Standard Edition AG         | “Secondary can be read”                   | Basic AG secondary isn't for readable workload offloading. ([Microsoft Learn][9])                                 |
| Bidirectional replication   | “Use updating subscribers” as the default | Legacy updatable subscriptions should be avoided for new development. ([Microsoft Learn][10])                     |
| Regional security           | “Dynamic Data Masking”                    | Use **Row-Level Security** for row isolation; DDM can supplement column protection. ([Microsoft Learn][13])       |
| Performance troubleshooting | “Use Profiler”                            | Prefer **Query Store + DMVs + Extended Events**; Profiler/Trace are deprecated. ([Microsoft Learn][6])            |
| Transactional replication   | “Two-way replication”                     | Normal transactional replication is Publisher → Subscriber.                                                       |
| Transactional articles      | “Any table”                               | Published transactional tables require a **primary key**. ([Microsoft Learn][3])                                  |
| Snapshot replication PK     | “PK mandatory”                            | Snapshot replication doesn't require a PK. ([Microsoft Learn][5])                                                 |
| Archiving                   | “Just partition the table”                | Start with retention/dependencies/access requirements, then select partitioning/archive DB/etc.                   |
| Azure migration             | “Backup and restore” immediately          | First choose **Azure SQL DB vs MI vs Azure VM**, assess, then choose migration technology. ([Microsoft Learn][1]) |
| 5-GB migration              | “Only bandwidth matters”                  | Compatibility, dependencies, security, connectivity, cutover and performance matter too.                          |
| Professional Edition        | Assume what interviewer means             | Verify `SERVERPROPERTY('Edition')`.                                                                               |
| Missing indexes             | Immediately create them                   | Validate workload + actual plan + Query Store + testing first.                                                    |

The **highest-priority subjects to prepare from this interview** are SQL Server replication, Always On/Basic AG limitations, Azure migration architecture, SQL performance troubleshooting, data archiving/retention, Row-Level Security vs Dynamic Data Masking, and production change management. Those themes account for most of the technical depth in the interview.  

[1]: https://learn.microsoft.com/en-us/data-migration/sql-server/managed-instance/guide?utm_source=chatgpt.com "SQL Server to Azure SQL Managed Instance: Migration Guide | Microsoft Learn"
[2]: https://learn.microsoft.com/en-us/sql/relational-databases/replication/snapshot-replication?view=sql-server-ver17&utm_source=chatgpt.com "Snapshot Replication - SQL Server | Microsoft Learn"
[3]: https://learn.microsoft.com/en-us/sql/relational-databases/replication/publish/publish-data-and-database-objects?view=sql-server-ver17&utm_source=chatgpt.com "Publish Data and Database Objects - SQL Server | Microsoft Learn"
[4]: https://learn.microsoft.com/en-us/sql/relational-databases/replication/administration/enhance-general-replication-performance?view=sql-server-ver17&utm_source=chatgpt.com "Enhance General Replication Performance - SQL Server | Microsoft Learn"
[5]: https://learn.microsoft.com/en-us/sql/relational-databases/replication/administration/frequently-asked-questions-for-replication-administrators?view=sql-server-ver17&utm_source=chatgpt.com "Frequently Asked Questions for Replication Administrators - SQL Server | Microsoft Learn"
[6]: https://learn.microsoft.com/en-us/sql/relational-databases/performance/best-practice-with-the-query-store?view=sql-server-ver17&utm_source=chatgpt.com "Best Practices for Monitoring Workloads with Query Store - SQL Server | Microsoft Learn"
[7]: https://learn.microsoft.com/en-us/troubleshoot/sql/database-engine/performance/understand-resolve-blocking?utm_source=chatgpt.com "Understand and resolve blocking problems - SQL Server | Microsoft Learn"
[8]: https://learn.microsoft.com/en-us/data-migration/sql-server/managed-instance/overview?utm_source=chatgpt.com "SQL Server to Azure SQL Managed Instance: Migration Overview | Microsoft Learn"
[9]: https://learn.microsoft.com/en-us/sql/linux/business-continuity/availability-groups/overview?view=sql-server-ver17&utm_source=chatgpt.com "Availability Groups for SQL Server on Linux - SQL Server | Microsoft Learn"
[10]: https://learn.microsoft.com/en-us/sql/relational-databases/replication/transactional/updatable-subscriptions-for-transactional-replication?view=sql-server-ver17&utm_source=chatgpt.com "Updatable Subscriptions (Transactional) - SQL Server | Microsoft Learn"
[11]: https://learn.microsoft.com/en-us/sql/relational-databases/replication/mssql-eng002627?view=sql-server-ver17&utm_source=chatgpt.com "MSSQL_ENG002627 - SQL Server | Microsoft Learn"
[12]: https://learn.microsoft.com/en-us/sql/relational-databases/replication/transactional/transactional-articles-specify-how-changes-are-propagated?view=sql-server-ver17&utm_source=chatgpt.com "Specify how changes are propagated (Transactional) - SQL Server | Microsoft Learn"
[13]: https://learn.microsoft.com/en-us/sql/relational-databases/security/dynamic-data-masking?view=sql-server-ver17&utm_source=chatgpt.com "Dynamic Data Masking - SQL Server | Microsoft Learn"
