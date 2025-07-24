# Scaling Estimations

Estimating the **Concurrency**, **TPS**, **Scaling Needs**, and perform **Back-of-the-Envelope (BOTE)** estimations for scale.

> Note: This project is a simplified prototype developed solely for educational purposes. The estimates provided are based on highly vague and undefined requirements. A fully developed platform of this nature would likely scale by a factor of 10 to 100 times or more.

## 1. **Concurrency & TPS Estimates**

### From Functional Specification Assumptions

* **User Types**: Public users, registered users, admins, auditors.
* **User Base**: Assume growth from a few hundred to **100,000+ users**.
* **Initial TPS**: 1 (as per earlier assumption).
* **Peak TPS**: 50-100 (as per earlier assumption).

| **User Type**    | **Estimated Concurrent Users (Peak)** | **Actions**                           | **Frequency**          |
| ---------------- | ------------------------------------- | ------------------------------------- | ---------------------- |
| Public Users     | 1,000                                 | Search data via public API            | High (frequent search) |
| Registered Users | 2,000                                 | Profile mgmt, search via private API  | Moderate               |
| Admins           | 5–10                                  | Upload, validate, approve breach data | Low                    |
| Auditors         | 10–20                                 | Read-only access, data approval       | Low                    |

> **Peak Concurrent Searches**: Assume 50% of 3,000 active users are searching simultaneously → \~1,500 search TPS bursts.
> Realistic averaged out over time = **50–100 TPS** under peak load.


## 2. **System Scaling Requirements**

### Read Path (User Search API)

* **Reads are high**: Must scale horizontally.
* Redis + PostgreSQL Read Replica(s) are a must.
* **Auto-scaling APIs** behind ALB.
* Optimize with:
  * k-anonymity cache in Redis (SHA-1 prefix → breach set).
  * Preindexed queries in PostgreSQL (on email, username).

### Write Path (Admin Data Ingestion)

* **Writes are lower**: Admin upload, approve, process jobs.
* Can be queue-based or batch processed.

## 3. Back-of-the-Envelope Calculations (BOTE)

Let’s assume you are targeting:

* **10 million breach records**
* **100,000 users**
* **Peak TPS = 100**
* **Average response time goal = <200ms**

### a. Storage Estimate

| **Component**                    | **Size per Record (avg)** | **Count**    | **Estimated Size**                  |
| -------------------------------- | ------------------------- | ------------ | ----------------------------------- |
| Breach Record (user, pass, etc.) | \~500 bytes               | 10,000,000   | \~5 GB raw (likely 2x with indexes) |
| Search Logs                      | \~200 bytes               | 100M entries | \~20 GB (for full audit logging)    |
| User Profiles                    | \~1 KB                    | 100,000      | \~100 MB                            |
| Admin Metadata                   | \~2 KB                    | 1,000        | \~2 MB                              |
| **Total Primary DB Size**        |                           |              | \~30–40 GB (compressed, indexed)    |
| **Raw Files (CSV/JSON logs)**    | Varies                    | 5,000 files  | \~10–50 GB in S3                    |

> **Use PostgreSQL for structured data + S3 for raw files.**

### b. Compute Estimate

| **Tier**           | **Instance Type**     | **Est. #** | **Purpose**                        |
| ------------------ | --------------------- | ---------- | ---------------------------------- |
| API Gateway + LB   | ALB                   | 1          | Entry point for all traffic        |
| Web/API Servers    | t3.medium (2vCPU/4GB) | 4–6        | Scales with traffic (Auto-Scale)   |
| DB Primary         | db.t3.large           | 1          | Admin writes, user profiles        |
| DB Read Replicas   | db.t3.medium          | 2–3        | User search queries                |
| Redis Cache        | cache.t3.micro        | 1–2        | Prefix query and metadata caching  |
| Background Workers | ECS Fargate or EC2    | 2–3        | Async ingestion, de-dupe, validate |

### c. Estimated Performance Benchmarks

| **Functionality**    | **Latency Goal** | **Implementation Notes**              |
| -------------------- | ---------------- | ------------------------------------- |
| Email Breach Search  | < 200 ms         | Redis cache or indexed DB lookup      |
| Password Pwned Check | < 100 ms         | SHA-1 prefix → Redis or indexed table |
| Admin Upload         | Async, <10 sec   | Chunked, background-processed         |
| Dashboard Metrics    | < 500 ms         | Pre-aggregated via cron/worker jobs   |
| Role/Permission Mgmt | < 300 ms         | Single-user low-volume flow           |

### 4. Summary of Scaling Plan

| **Metric**           | **Early Stage**          | **Scaling Plan**                                           |
| -------------------- | ------------------------ | ---------------------------------------------------------- |
| Total Breach Records | 100K → 10M               | PostgreSQL sharding (domain or hash), use S3 + metadata DB |
| Peak Users           | 100 → 100,000            | Use auto-scaling groups, CDN for static, Redis for caching |
| TPS (avg/peak)       | 1 → 50 (avg), 100 (peak) | Horizontally scale API layer; separate read/write models   |
| Availability Goal    | 99.9%                    | Multi-AZ, failover DBs, health checks                      |
| Storage Footprint    | 5 GB → 50+ GB            | PostgreSQL RDS, lifecycle-managed S3                       |
| API Latency Target   | ≤200 ms                  | Redis cache + DB tuning + connection pooling               |
