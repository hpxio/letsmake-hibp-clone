# Functional Specification Document (FSD)

## 🎯 1. Purpose and Goals

Develop clone of HaveIBeenPawnd (HIBP) for learning system design in-depth.

### 💠 Business Context
The service like HIBP, collects and analyzes hundreds of database dumps and pastes containing information about billions of leaked accounts, and allows users to search for their own information by entering their username or email address.

### 💠 Objectives
This is entirely for learning. HIBP type service potentially have huge database ready to be accessed by numerous users concurrently. It has multiple types of users which perform different set of operations. Following are the potential system-design aspects I am expecting to learn from this.

📒 **Large Data Organization & Storage**
- Designing data schemas to efficiently store millions of records (e.g., email, phone, breach metadata).
- Evaluating trade-offs between SQL (e.g., PostgreSQL) vs NoSQL (e.g., ElasticSearch, MongoDB) for search-heavy use cases.

📒 **Search & Indexing (Fast Lookup)**

- Implementing search functionality for fast querying by email or phone.
- Indexing strategies to optimize performance on frequent queries.

📒 **CQRS (Command Query Responsibility Segregation)**

- Learning to separate read and write operations:
    - Write side: Admin ingestion of breach data.
    - Read side: End-user querying/checking breaches.
- Understanding how to keep read models updated asynchronously (eventual consistency).

📒 **Caching Strategies**

- Caching frequent queries using Redis or similar in-memory stores.
- Preventing backend overload with aggressive caching of non-sensitive and high-traffic pages (e.g., breach details, known breaches list).

📒 **Security Best Practices**

- Hashing and k-anonymity for privacy-preserving lookup (e.g., like HIBP's k-anonymity model with SHA-1 prefix matching).
- Rate limiting and abuse detection to prevent automated data scraping.

📒 **Admin Data Management**

- Admin dashboard to upload breach dumps, validate data format, and process entries.
- Background jobs for data ingestion and indexing.
- Tagging metadata like breach source, date, affected services, and data types.

📒 **API Design**

- RESTful or GraphQL APIs for client-side interaction.
- Public-facing APIs with throttling, paging and API keys.
- Internal APIs for admin data ingestion.

📒 **Scalability & Load Handling**

- Strategies for horizontal scaling of read replicas.
- Load balancing and auto-scaling policies on cloud platforms.
- Estimating query volumes and designing the system to scale accordingly.

📒 **Deployment & Monitoring**

- Containerization using Docker, deployment on cloud platforms (e.g., Heroku, AWS, Vercel).
- Logging, metrics, and uptime monitoring.
- Using tools like Prometheus, Grafana, or ELK stack for observability.

### 💠 Success Criteria
  - _TBD_


## 🎯 2. Feature Scope

### 💠 In Scope

List of features that are planned for current development.

#### ⭐️ Feature-1: Data Ingestion & Organization

- **Data Sourcing Interface** – Accept CSV, JSON, or DB dump.
- **Schema Definition & Validation** – Email, phone, service name, breach date, data types affected.
- **Data Sanitization** – Strip irrelevant fields, PII redaction.
- **Metadata Annotation** – Tag with origin (dark web, public post), date, sensitivity.
- **Data Hashing / Encryption** – SHA-1 (for k-anonymity), or bcrypt for passwords.

#### ⭐️ Feature-2: Admin APIs & Dashboard

- **Authentication & Role Management** – Admin-only access (JWT or OAuth-based).
- **Breach Upload Interface** – API to upload datasets.
- **Glance Dashboard API** – Active breaches, ingestion jobs status, errors.
- **DB Health & Storage View** – Available storage, ingestion queue length, DB load.
- **Audit Logs** – Track who uploaded/modified what and when.

#### ⭐️ Feature-3: Search Engine & Public Query API

- **Search by Data Model** – Exact match, partial match (with k-anonymity).
- **Search by Breach Metadata** – Domain, type (passwords, PII), source.
- **Rate-Limited Public API** – With standard throttle rules.
- **User Access Security** – RBAC Security, JWT/OAUth for Users

#### ⭐️ Feature-4: Abuse Prevention, Fair Use & Subscription

- **Rate Limiting & Abuse Detection** – IP throttling, query frequency tracking.
- **Free vs Paid API Plans** – Token-based usage; free tier vs enterprise tier.
- **Subscription Billing Integration** – Stripe, LemonSqueezy, or manual keys.
- **User API Keys & Quotas** – API management dashboard.
- **Privacy Preserving Techniques** – k-anonymity, obfuscated\* identifiers.


#### ⭐️ Feature-5: Testing, Quality, and Security

- **Automated Unit & Integration Tests**
- **Load Testing & Benchmarking** – Simulate high search volumes.
- **Vulnerability Scanning** – Static analysis, dependency audits (e.g., Snyk).
- **Secure Deployment Pipelines** – Secrets rotation, GitHub Actions with OIDC.

### Out Scope

List of features that are planned for later.

#### ⭐️ Feature-0: Admin Audit Log for Users (Future)

#### ⭐️ Feature-1: Notifications & User Engagement (Future)

- **User Watchlist / Alerts** – Notify registered user’s email appears in new breach.
- **Opt-in Alert System** – Email or browser push (notify when breach found)


## 🎯 3. User Personas

### 💠 User Roles

| Role            | **Description**                                                               | **Primary Goals**                                         | Permissions                                               |
| --------------- | ----------------------------------------------------------------------------- | --------------------------------------------------------- | --------------------------------------------------------- |
| **Admin**       | Manages breach data, users, and system settings.                              | - Configure system <br> - Upload and manage breach data   | - Full access to all features.                            |
| **Auditor**     | Views and validates breach data for compliance and integrity.                 | - Ensure compliance <br> - Verify breach authenticity     | - Read-only access to all breach records.                 |
| **User/Guest**  | Searches breach data and receives notifications about personal data breaches. | - Check personal data <br> - Stay informed about breaches | - Can query breaches by email/username.                   |
| **Super Admin** | Manages other admins, defines roles, and monitors system performance.         | - Role management <br> - Oversee all configurations       | - Has control over all admin accounts and configurations. |


### 💠 User Permissions (_TBD_)

| Feature                   | Guest | Users | Auditor | Admin | Super Admin |
| ------------------------- | ----- | ----- | ------- | ----- | ----------- |
| Data Loading              | ❌     | ❌     | ❌       | ✅     | ✅           |
| Data Auditing             | ❌     | ❌     | ✅       | ✅     | ❌           |
| Data Approval             | ❌     | ❌     | ✅       | ✅     | ❌           |
| Admin Dashboard           | ❌     | ❌     | ✅       | ✅     | ✅           |
| User Management           | ❌     | ❌     | ❌       | ✅     | ✅           |
| Role Management           | ❌     | ❌     | ❌       | ❌     | ✅           |
| System Monitoring         | ❌     | ❌     | ❌       | ❌     | ✅           |
| System Configurations     | ❌     | ❌     | ❌       | ❌     | ✅           |
| View User Profile         | ❌     | ✅     | ❌       | ✅     | ❌           |
| Create User Profile       | ❌     | ✅     | ❌       | ✅     | ❌           |
| Update User Profile       | ❌     | ✅     | ❌       | ✅     | ❌           |
| Delete User Profile       | ❌     | ✅     | ❌       | ✅     | ❌           |
| Self-profile Management   | ❌     | ✅     | ✅       | ✅     | ✅           |
| Search Data (Private API) | ❌     | ❌     | ✅       | ✅     | ✅           |
| Search Data (Public API)  | ✅     | ✅     | ❌       | ✅     | ✅           |


## 🎯 5. Use Cases & User Stories

See Use Cases document.

## 🎯 6. Functional Requirements

| **Module**                | **Functional Requirement**                                                            | **Description**                                                                 |
| ------------------------- | ------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| **User Search**           | Allow users to search breach data by email or username.                               | Users can query the system to check if their data has been exposed in a breach. |
|                           | Provide a password pwned lookup service using hashed input (k-anonymity).             | Users can check if a password is compromised without exposing it.               |
|                           | Return breach details like breach name, source domain, and data types exposed.        | Search results will include metadata about the breach for user awareness.       |
| **Admin Services**        | Enable admins to upload breach data in CSV, JSON, or log formats.                     | Admins can ingest new breach data into the system securely.                     |
|                           | Support role-based access control (viewer, editor, admin).                            | Admins have restricted access based on roles to ensure security.                |
|                           | Provide an interface to manage breaches (update, delete, view stats).                 | Admins can perform CRUD operations on breach data and view analytics.           |
| **Data Ingestion**        | Parse and validate uploaded data files for consistency.                               | Ensure only clean and valid data is ingested into the system.                   |
|                           | Deduplicate records based on email/username.                                          | Remove redundant entries to optimize storage and accuracy.                      |
|                           | Store raw breach data in AWS S3 for archival purposes.                                | Retain original files for later verification or processing.                     |
| **Notification Service**  | Notify subscribed users when their data appears in a new breach.                      | Automated emails or push notifications sent on detecting matches.               |
|                           | Allow users to opt-in or opt-out of notifications.                                    | Ensure GDPR/CCPA compliance with user preferences.                              |
| **Analytics & Reporting** | Log all user searches with metadata like time and result count.                       | Collect data for system monitoring and performance evaluation.                  |
|                           | Provide breach statistics to admins (e.g., most common breaches, data types exposed). | Offer visual insights to admins for better understanding of breach trends.      |
| **Authentication**        | Implement user authentication for search and admin services.                          | Secure system access using OAuth2 or JWT-based mechanisms.                      |
|                           | Enforce strong password policies for admin accounts.                                  | Prevent brute force and other common attack vectors.                            |
|                           | Support email/password login for users and admins.                                    | A straightforward and secure authentication mechanism.                          |


## 9. Non-Functional Requirements

In this project I am targetting Availability & Partition Tolerance. The NFRs are derived based accordingly.

* Why Availability and Partition Tolerance (AP)?
  * Prioritizing high availability for user searches. Users should get a response even if a shard or node is down.
  * Perfect consistency is less critical for user searches. Slight delays in reflecting new breach data are acceptable.
* Partition tolerance is a must for scalability across multiple nodes or regions.        

| **NFR**                        | **Implementation Suggestions**                                                                                                                                              |
| ------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Scalability**                | - **Horizontal Scaling**: Use AWS Auto Scaling for EC2 instances. <br> - **Load Balancing**: Integrate AWS Application Load Balancer (ALB) to distribute user traffic efficiently. |
| **Availability**               | - Ensure a minimum uptime of **99.9%**. <br> - Use AWS **Multi-AZ** deployments for RDS and redundant S3 buckets.                                                                  |
| **Latency**                    | - Use **ElastiCache** (Redis/Memcached) for caching frequent queries (e.g., user search results). <br> - Leverage AWS CloudFront as a CDN for faster content delivery.             |
| **Security**                   | - Enforce **HTTPS** using AWS ACM for certificates. <br> - Implement IAM roles and policies with least privilege access. <br> - Use AWS WAF to prevent DDoS attacks.               |
| **Monitoring and Alerts**      | - Use AWS CloudWatch, ELK Stack, Prometheus & Grafana for monitoring system health. <br> - Set up alarms for high latency, low memory, or unusual traffic.                         |
| **Disaster Recovery**          | - Regularly back up data using **AWS Backup**. <br> - Design RPO < 5 minutes and RTO < 15 minutes.                                                                                 |
| **Cost Optimization**          | - Choose **T3/T4a instances** for cost efficiency during early phases. <br> - Use **S3 lifecycle policies** for data archival.                                                     |
| **Logging**                    | - Integrate centralized logging using AWS CloudWatch Logs or **ELK stack** (ElasticSearch, Logstash, Kibana).                                                                      |
| **Data Retention and Purging** | - Define a policy (e.g., keep 2 years of data). <br> - Automate purging with AWS Lambda or periodic database scripts.                                                              |

_NOTE_ Cost estimations are left, I will add it little later. (2025-06-23)


## 7. UI/UX Design
- _TBD_


## 10. Reporting & Analytics Requirements
- Event tracking for login/logout
- Dashboard usage metrics

## 11. Assumptions & Constraints
- Users must have valid email addresses
- Internet connectivity required

## 12. Known Limitations / Deferred Items
- No multi-language support in phase 1
- Role-based dashboards planned for phase 2
