# Data Model

## Source Data Structure

For the sake of simplicity & to focus on learning, I will use fabricated data with random fake-data-generator. 
To keep source data file structure simple, I am focusing on following input types:

### **File Types**

1. **CSV**: Tabular data for easy parsing and storage.
2. **JSON**: Nested data for flexibility in structure.
3. **Stealer Logs**: Unstructured or semi-structured text mimicking stealer dumps.

#### **Probable Fields in CSV/JSON**

| **Field**        | **Description**                                                            |
| ---------------- | -------------------------------------------------------------------------- |
| **Email**        | Email addresses of users involved in breaches.                             |
| **Username**     | Associated usernames, if available.                                        |
| **Password**     | Passwords in plaintext (for stealer logs) or hashed format (for CSV/JSON). |
| **Domain**       | Breach source domain or associated service.                                |
| **Address**      | Fabricated physical addresses (optional for some breaches).                |
| **Phone Number** | Associated phone numbers (optional for some breaches).                     |


#### **Sample Stealer Log**

  ```
  Username: user123
  Password: plain_password
  Email: user@example.com
  Domain: example.com
  ```

---

## **Data Model & Entities**

Based on my requirements and fabricated data fields, here’s the initial draft of the **data model** with corresponding entities:

#### **Core Entities**

| **Entity**       | **Attributes**                                                                                             | **Description**                                                                         |
| ---------------- | ---------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------- |
| **Breach**       | `id`, `name`, `source_domain`, `breach_date`, `data_type`, `description`, `isVerified`, `isMarkedArchived` | Represents a data breach event. Includes metadata about the breach.                     |
| **BreachRecord** | `id`, `email`, `username`, `hashed_password`, `address`, `phone_number`, `breach_id (FK)`                  | Stores individual records linked to a breach. Optimized for searches by email/username. |
| **Admin**        | `id`, `username`, `email`, `role`, `password_hash`                                                         | Stores admin credentials and role-based access levels.                                  |

_NOTE_: More entities will be added later as the design evolves.

### **Entity Relationships**

* A **Breach** has many **BreachRecords** (1-to-Many).
* **Admin** has role-based access control (`ROLE_VIEWER`, `ROLE_EDITOR`, `ROLE_ADMIN`).

---

## Proposed Database Architecture

### Primary Database for Relational Data

| **Database**            | **Why?**                                                                                                     |
| ----------------------- | ------------------------------------------------------------------------------------------------------------ |
| **PostgreSQL**          | - Supports complex queries, indexing, and relationships efficiently. <br> - Open-source and highly reliable. |
| **MySQL** (alternative) | - Similar to PostgreSQL; use if your team is already experienced with it.                                    |

* **Structure**: Relational database for structured data like breaches, records, and logs.
* **Indexes**: Create indexes on frequently queried fields like `email`, `username`, and `domain`.

#### **Secondary Store for Fast Lookup**

| **Database**            | **Why?**                                                                                                   |
| ----------------------- | ---------------------------------------------------------------------------------------------------------- |
| **ElastiCache (Redis)** | - Ideal for caching frequent queries like `email` lookups. <br> - Provides sub-millisecond response times. |

* **Use Case**: Cache breach lookup results to reduce load on the primary database.
* **Eviction Policy**: Use an LRU (Least Recently Used) policy to maintain relevant data in cache.

#### **Blob Storage for Raw Data**

| **Storage** | **Why?**                                                                                                          |
| ----------- | ----------------------------------------------------------------------------------------------------------------- |
| **AWS S3**  | - Scalable, cost-effective, and secure. <br> - Supports lifecycle policies for archival and deletion of old data. |

* **Use Case**: Store raw breach files (CSV, JSON, stealer logs) before ingestion.


### **Why NoSQL is Unnecessary Here**

1. **Data Structure Complexity**:

   * Data has a relational structure (e.g., breach records linked to breaches), which fits well into relational databases.
   * NoSQL excels at unstructured or semi-structured data but would add complexity for relational queries.

2. **Scalability**:

   * PostgreSQL supports horizontal scaling with sharding, which addresses my need for scaling without requiring NoSQL.

3. **Consistency**:

   * Relational databases provide strong consistency (ACID compliance), which is critical for breach data integrity.

4. **Querying Needs**:

   * SQL's querying power is more suited for my use case (e.g., searching by email, domain, or username).

5. **Cost and Complexity**:

   * Introducing NoSQL increases the complexity of your system without significant gains for your workload.


### FR/NFR Justifications
* Scaling/Storage
* Cost Concerns
* Security


## Proposed Storage Tiering

| **Tier**          | **Storage Type**        | **Purpose**                                                                             | **Example Data**                                |
| ----------------- | ----------------------- | --------------------------------------------------------------------------------------- | ----------------------------------------------- |
| **Hot Tier**      | Redis (ElastiCache)     | For frequently queried data like `email` and `username`. Ensures low-latency responses. | Cached breach lookup results.                   |
| **Warm Tier**     | PostgreSQL (Primary DB) | For structured, relational data requiring consistent queries.                           | Breach records, admin controls, search logs.    |
| **Cold Tier**     | AWS S3                  | For storing raw breach files and archival data.                                         | Original breach files (CSV, JSON, logs).        |
| **Archived Tier** | AWS Glacier             | For old, infrequently accessed data to reduce storage costs.                            | Historical breach records older than 2-3 years. |

### **Data Movement Between Tiers**

* Frequently accessed data moves from PostgreSQL to Redis (cache).
* Older, less-accessed data can be offloaded from PostgreSQL to S3 or Glacier for cost savings.

