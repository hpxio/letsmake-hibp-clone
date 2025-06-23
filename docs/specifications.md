# 🎯 Requirements Specifications

## ⚙️ Functional Requirements
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


## ⚙️ Non-functional Requirements

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


## ⚙️ Service Architecture


| **Microservice**           | **Responsibilities**                                                                                            | **Tech Stack**                                                                                  |
| -------------------------- | --------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| **User Service**           | - Allow users to search for breaches using email/phone. <br> - Provide password pwned lookup using k-anonymity. | - **Spring Boot**, AWS RDS (PostgreSQL/MySQL), Redis (for caching), AWS API Gateway*.           |
| **Admin Service**          | - Enable breach data upload and management. <br> - Role-based access control for 3 admin levels.                | - **Spring Boot**, AWS RDS, Spring Security Module.                                             |
| **Breach Data Service**    | - Handle ingestion, cleaning, and indexing of fabricated breach datasets.                                       | - **Spring Boot**, AWS S3 (for storage)                                                         |
| **Notification Service**   | - Send notifications to users about breaches.                                                                   | - **Spring Boot**, Java SMTP, Apache Kafka for Message Queuing                                  |
| **Authentication Service** | - Provide secure authentication and authorization for users and admins.                                         | - **Spring Boot** Spring Security Module with OAuth2 or JWT                                     |


## ⚙️ Tech Stack Overview


| **Category**                 | **Suggested Tech Stack**                                                                         |
| ---------------------------- | ------------------------------------------------------------------------------------------------ |
| **Backend Framework**        | Spring Boot for all microservices.                                                               |
| **Database**                 | AWS RDS with PostgreSQL or MySQL.                                                                |
| **Caching**                  | ElastiCache (Redis) for fast lookups.                                                            |
| **Data Storage**             | AWS S3 for breach datasets and backups.                                                          |
| **Message Queue (Optional)** | Apache Kafka for asynchronous processing (e.g., notifications, breach data ingestion).           |
| **Authentication**           | Spring Security + AWS Cognito for OAuth2/JWT-based authentication and role-based access control. |
| **Deployment**               | Docker + ECS (or Kubernetes, if preferred).                                                      |
| **Monitoring**               | AWS CloudWatch for logs, metrics, and alerts.                                                    |
| **Frontend (Optional)**      | React or Angular for admin dashboards (if needed).                                               |
