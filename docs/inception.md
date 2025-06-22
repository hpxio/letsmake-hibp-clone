# Clone - HaveIBeenPawned

# ⭐️ Objective

"I want to host information of breached data and provide a convenient interface for accessing information in a structured manner to the client."

- Client is the end-user who wants to check whether their information is breached.
- Here, breach means any information of the user like email, phone number, address, etc. that are stolen & made public in a security incident.

  
I want to build a platform-clone, much like HaveIBeenPwned (HIBP), that allows users to check if their email addresses or phone numbers have been exposed in publicly known data breaches. If a user finds their email or phone number on the site, it means that it was part of a database that was compromised in a past security incident.  


---

## ⭐️ How Data Is Collected

The service like HIBP, collects and analyzes hundreds of database dumps and pastes containing information about billions of leaked accounts, and allows users to search for their own information by entering their username or email address. Hence, HIBP does not own the data itself but rather depend on public sources which are often reliable and verifiable. I will follow same model while building the clone.  


## ⭐️ Meaning of "Pwned"

The term has been adopted in the context of online security and refers to a situation where a user's account or system has been compromised. This could involve unauthorized access to an email account, a social media profile, or any other online service  


---

## ⭐️ Learning Scope

This is entirely for learning. Following are the potential system-design aspects I am expecting to learn from this.

#### 1\. **Large Data Organization & Storage**

- Designing data schemas to efficiently store millions of records (e.g., email, phone, breach metadata).
- Evaluating trade-offs between SQL (e.g., PostgreSQL) vs NoSQL (e.g., ElasticSearch, MongoDB) for search-heavy use cases.
- Understanding normalization vs denormalization in the context of breach records.

#### 2\. **Search & Indexing (Fast Lookup)**

- Implementing search functionality for fast querying by email or phone.
- Use of search engines (e.g., ElasticSearch) for full-text and keyword search.
- Indexing strategies to optimize performance on frequent queries.

#### 3\. **CQRS (Command Query Responsibility Segregation)**

- Learning to separate read and write operations:
    - Write side: Admin ingestion of breach data.
    - Read side: End-user querying/checking breaches.

- Understanding how to keep read models updated asynchronously (eventual consistency).

#### 4\. **Caching Strategies**

- Caching frequent queries using Redis or similar in-memory stores.
- Preventing backend overload with aggressive caching of non-sensitive and high-traffic pages (e.g., breach details, known breaches list).
- Using CDN to cache static pages and rate-limiting abusive requests.

#### 5\. **Security Best Practices**

- Secure input handling (e.g., to prevent enumeration attacks).
- Hashing and k-anonymity for privacy-preserving lookup (e.g., like HIBP's k-anonymity model with SHA-1 prefix matching).
- Rate limiting and abuse detection to prevent automated data scraping.

#### 6\. **Admin Data Management**

- Admin dashboard to upload breach dumps, validate data format, and process entries.
- Background jobs for data ingestion and indexing.
- Tagging metadata like breach source, date, affected services, and data types.

#### 7\. **API Design**

- RESTful or GraphQL APIs for client-side interaction.
- Public-facing APIs with throttling and API keys.
- Internal APIs for admin data ingestion.

#### 8\. **Scalability & Load Handling**

- Strategies for horizontal scaling of read replicas.
- Load balancing and auto-scaling policies on cloud platforms (e.g., AWS/GCP).
- Estimating query volumes and designing the system to scale accordingly.

#### 9\. **User Privacy & Ethics**

- Privacy-preserving designs to ensure users are not identifiable by others.
- Educating users without scaring them—striking a balance between transparency and security awareness.

#### 10\. **Deployment & Monitoring**

- Containerization using Docker, deployment on cloud platforms (e.g., Heroku, AWS, Vercel).
- Logging, metrics, and uptime monitoring.
- Using tools like Prometheus, Grafana, or ELK stack for observability.

---

## ⭐️ Features

List of all features I can think of now. And followed by that is list of phase-wise execution of each features.

#### **✓ Feature 1: Data Ingestion & Organization**

- **Data Sourcing Interface** – Accept CSV, JSON, or DB dump.
- **Schema Definition & Validation** – Email, phone, service name, breach date, data types affected.
- **Data Sanitization** – Strip irrelevant fields, PII redaction.
- **Metadata Annotation** – Tag with origin (dark web, public post), date, sensitivity.
- **Data Hashing / Encryption** – SHA-1 (for k-anonymity), or bcrypt for passwords.

#### **✓ Feature 2: Admin APIs & Dashboard**

- **Authentication & Role Management** – Admin-only access (JWT or OAuth-based).
- **Breach Upload Interface** – API to upload datasets.
- **Glance Dashboard API** – Active breaches, ingestion jobs status, errors.
- **DB Health & Storage View** – Available storage, ingestion queue length, DB load.
- **Audit Logs** – Track who uploaded/modified what and when.

#### **✓ Feature 3: Search Engine & Public Query API**

- **Search by Data Model** – Exact match, partial match (with k-anonymity).
- **Search by Breach Metadata** – Domain, type (passwords, PII), source.
- **Rate-Limited Public API** – With standard throttle rules.
- **User Access Security **– RBAC Security, JWT/OAUth for Users

#### **✓ Feature 4: Abuse Prevention, Fair Use & Subscription**

- **Rate Limiting & Abuse Detection** – IP throttling, query frequency tracking.
- **Free vs Paid API Plans** – Token-based usage; free tier vs enterprise tier.
- **Subscription Billing Integration** – Stripe, LemonSqueezy, or manual keys.
- **User API Keys & Quotas** – API management dashboard.
- **Privacy Preserving Techniques** – k-anonymity, obfuscated\* identifiers.

#### **✓ Feature 5: Notifications & User Engagement (Future)**

- **User Watchlist / Alerts** – Notify registered user’s email appears in new breach.
- **Opt-in Alert System** – Email or browser push (notify when breach found)

#### **✓ Feature 6: Testing, Quality, and Security**

- **Automated Unit & Integration Tests**
- **Load Testing & Benchmarking** – Simulate high search volumes.
- **Vulnerability Scanning** – Static analysis, dependency audits (e.g., Snyk).
- **Secure Deployment Pipelines** – Secrets rotation, GitHub Actions with OIDC.

---

## ⭐️ Execution Plan

#### **🎯 Phase 0 – Planning & Setup**

- Architecture Blueprint
- Infrastructure & Costing Plan
- Tech Stack Finalization
- Code Repository, CI/CD, Monitoring setup
- Region & Market Definition

#### **🎯 Phase 1 – MVP**

Core goal: **Working platform that allows basic breach lookup**

- **Feature 1**: Data Ingestion (manual upload, parsing, sanitizing, hashing)
- **Feature 2**: Admin API (basic upload + dashboard view)
- **Feature 3**: Basic Search API (email/phone direct match)
- **Security & Rate Limiting Basics**

#### **🎯 Phase 2 – Enhanced Capability**

Core goal: **Improve privacy, extensibility & user control**

- Add k-anonymity-based search for passwords
- Add metadata-based search (by service, type)
- Improve admin dashboard (with data stats, retry ingestion, logs)
- Introduce basic usage quotas & monitoring
- Start prepping notification infrastructure

#### **🎯 Phase 3 – Productization**

Core goal: **Support real usage & prepare for growth**

- Add subscriptions & API key management
- Introduce user watchlists & breach alert system
- CDN cache integration, scalable DB + search setup
- Abuse detection and auto-blacklisting

---

## ⭐️ Planning

### **🎯 **Strategic Planning

- **Architecture Blueprint** – Define system components (e.g., ingestion service, search service, API gateway).
- **Target Region & User Profiling** – Where will it launch? Who are your users?
- **Scale Forecasting** – Estimated user load, DB growth, API traffic.
- **Cost Estimation** – Infra, storage, egress, CDN, logging, etc.
- **Tech Stack Finalization** – Backend, frontend, DB, hosting, search engine, etc.
- **Infra Provider Selection** – AWS / GCP / Azure / Vercel / Cloudflare / Render
- **Security Compliance Goals** – GDPR readiness, data minimization principles.

### **🎯 **Development Planning

- **Detailed Engineering Plan -** HLD/LLD, DFD, DOD, ADR
- **Code Repository & Branch Strategy** – GitHub/GitLab; trunk-based or GitFlow.
- **Progress Tracking Tools** – Jira, Linear, Notion, GitHub Projects.
- **CI/CD Pipeline Setup** – Auto testing, deployments, linting, containerization.
- **Monitoring & Logging Stack** – Grafana, Prometheus, Sentry, Loki, etc.

---

## ⭐️ Data Definition

- Data Model (MVP Scope)
    - Pawnage Data : Phone Number, Email, Address, Passwords
    - Metadata : TBD



