# Database Essentials and Vulnerabilities

> [!NOTE]
> This document surveys database fundamentals and the security risks that commonly threaten them. It contrasts relational and NoSQL models, ER-based data modelling, and essential SQL/administration tasks (DDL/DML/DCL, roles, permissions, and backup strategies). It then covers data protection practices of classification, data at rest/in transit/in use, sovereignty, encryption, masking, tokenisation, segmentation, and auditing. Finally, it explains injection vulnerabilities (OS command, SQL, NoSQL, LDAP, XPath) and prevention aligned with OWASP and NIST guidance.
## Database Fundamentals
### Relational databases
- A relational database stores data in tables with rows for records and columns for attributes.
- Tables relate through shared fields, commonly an identifier such as Customer ID. Relationships enable joins that combine data from multiple tables into a new result set for reporting and analysis.
- Relational database management systems query data using SQL and enforce a defined schema. Columns can be constrained by data type and allowed values, improving consistency and integrity.
- Key benefits include reduced redundancy through normalised design, strong access control, and mature backup and recovery options, including continuous replication in many managed cloud services.
- ACID properties support reliable transactions: atomicity, consistency, isolation, and durability.
- Deployments range from desktop and on-premises servers to cloud database-as-a-service. Common products include IBM Db2, Microsoft SQL Server, MySQL, Oracle Database, and PostgreSQL, with managed offerings such as Amazon RDS, Google Cloud SQL, IBM Db2 on Cloud, Oracle Cloud databases, and Azure SQL Database.
### NoSQL databases
- NoSQL refers to non-relational databases, often not only SQL. They typically use flexible schemas that suit rapidly changing application data.
- Four common models are key-value, document, column-family, and graph.
- NoSQL systems are widely used for high-volume web and mobile workloads because they can scale out across many nodes and data centres on commodity hardware.
- Trade-offs vary by product. Compared with many relational systems, some NoSQL designs offer different consistency and transaction guarantees, and may be less suited to complex ad hoc joins.
### Data modelling with the ER approach
- The relational model is widely used because it supports logical and physical data independence, and separates data structure from storage details.
- An entity-relationship diagram describes a database using entities and their relationships. Entities represent independent objects, while attributes describe their properties.
- When implemented in a relational database, entities map to tables and attributes map to columns. Typical data types include CHAR and VARCHAR for text, numeric types for counts and amounts, and date and time types for timestamps.
- A primary key uniquely identifies each row in a table. A foreign key references a primary key in another table and defines how tables relate.
### Example schema checks
- In a simple Car Dealership schema, three relations are used: CAR, SALE, and SALESPERSON.
- CAR includes four columns: Serial_no, Model, Manufacturer, and Price.
- SALE can reference CAR and SALESPERSON using foreign keys. Serial_no in SALE can reference CAR.Serial_no, and Salesperson_id in SALE can reference SALESPERSON.Salesperson_id.
- Primary keys can be Serial_no for CAR and Salesperson_id for SALESPERSON, ensuring unique identification of vehicles and staff.
### Summary
- Relational databases suit structured data, OLTP systems with frequent inserts and updates, and many data warehouse and OLAP workloads.
- NoSQL databases suit large, diverse, or rapidly evolving data and distributed, high-scale applications where flexible models and horizontal scaling are priorities.
## SQL Basics and Database Management Fundamentals
### SELECT queries and result sets
- A SELECT statement retrieves rows from one or more tables and returns a result set.
- A query can return all columns with * or selected columns by name.
- Filtering is performed with a WHERE clause that contains a predicate, which is a condition that evaluates to true, false, or unknown.
- Common comparison operators are =, <, >, <=, >=, and <> for not equal.
- Character literals are normally enclosed in single quotes, for example CCODE = 'CA'.

```sql
select book_id, title
from book
where book_id = 'B1'
```
### Useful SELECT expressions
- COUNT returns the number of matching rows. COUNT(*) counts rows, while COUNT(column) counts non-NULL values.
- DISTINCT removes duplicate values from a result set.

```sql
select count(*)
from medals

select distinct country
from medals
where medaltype = 'GOLD'
```

- LIMIT restricts returned rows in many systems such as MySQL, PostgreSQL, and SQLite. Other platforms use equivalents such as TOP or FETCH FIRST.

```sql
select *
from medals
where year = 2018
limit 5
```
### Inserting and changing data
- INSERT adds new rows to a table. The number of values should match the listed columns.
- Multiple rows can be inserted by providing multiple value groups.

```sql
insert into author (author_id, lastname, firstname, email, city, country)
values
('A1', 'Chong', 'Raul', 'rfc@ibm.com', 'Toronto', 'CA'),
('A2', 'Ahuja', 'Rav', 'rav@ibm.com', 'Bangalore', 'IN')
```

- UPDATE changes existing rows. DELETE removes rows.
- The WHERE clause is critical because omitting it typically affects all rows in the table.

```sql
update author
set lastname = 'Katta', firstname = 'Lakshmi'
where author_id = 'A2';

delete from author
where author_id in ('A2','A3');
```
### SQL command categories and data movement
- Databases are commonly administered using SQL.
- Data definition language, DDL, defines schema objects such as databases and tables, for example CREATE, ALTER, DROP, and TRUNCATE.
- Data manipulation language, DML, reads and changes data, for example SELECT, INSERT, UPDATE, and DELETE.
- Data control language, DCL, manages access control, commonly GRANT and REVOKE. Some platforms such as Microsoft SQL Server also support DENY.
- Data can be loaded through INSERT and bulk import tools, and extracted through queries and reporting tools.
### Roles, permissions, and least privilege
- Permissions authorise actions at different scopes, typically database, system, and object levels, though terminology differs by platform.
- Roles group permissions so that access can be granted consistently to categories of users rather than to each user individually.
- Effective access usually reflects the union of permissions granted directly and through enabled roles, so role design should follow least privilege.
### Backup types and common strategies
- Backups protect against data loss and support recovery after failures or security incidents.
- Physical backups capture database files and logs, enabling full restoration of an instance when all required files are available.
- Logical backups export schema and data, supporting migration and selective restore, but they generally require a working target database engine for import.
- Common strategies include full backups, differential backups that capture changes since the last full backup, and incremental backups that capture changes since the last backup of any type.
- Backup management typically includes automation, monitoring, retention, and periodic restore testing to validate recovery.
## Database Security and Data Protection Strategies
### Data classification for database security
Data classification groups information by sensitivity and business impact so that organisations can apply proportionate controls and meet legal obligations.

- Sensitive data: information that could cause harm if disclosed, such as health details, financial records, and government identifiers. It requires strong encryption, strict access control, and monitoring.
- Confidential data: information intended to remain internal, such as trade secrets, client lists, and business strategies. It needs encryption, limited access, staff training, and contractual controls such as non-disclosure agreements.
- Restricted data: information available only to specific roles or teams, such as internal documents, proprietary research, and employee records. It should be protected with encryption, multi-factor authentication, and role-based access control, plus regular security reviews.
- Private data: personal information that individuals expect to remain private, such as contact details and private communications. It must be handled in line with privacy requirements and supported by access controls, encryption, and privacy impact assessments.
- Critical data: information essential to operations and organisational survival, such as key operational datasets, intellectual property, and core financial systems. It requires the highest level of protection, including strong backup and recovery, resilience planning, and continuous risk management.
- Public data: information intended for public release, such as marketing materials and public research. It does not require confidentiality controls, but still needs integrity and availability safeguards such as backups and change control.
### Data states, sovereignty, and geolocation
Data protection varies by how and where data exists.

- Data at rest: stored in databases, file systems, and backups. Controls include encryption at rest, access control, auditing, and data masking for non-production use.
- Data in transit: moving across networks and services. Controls include transport encryption such as TLS, secure channels such as VPNs, endpoint authentication, and integrity checks using hashes.
- Data in use: processed in memory by applications and operating systems. Controls include secure coding, access control, runtime monitoring, and specialised techniques such as confidential computing or encryption-aware processing where feasible.

Data sovereignty describes how legal requirements apply based on data location. Organisations commonly address sovereignty through data residency decisions, localisation requirements, and compliant cross-border transfer mechanisms such as standard contractual clauses and binding corporate rules, where applicable.

Geolocation can strengthen security by applying location-based policies, improving fraud detection, and supporting residency controls. It is often used to trigger additional verification for high-risk regions and to detect anomalous access patterns.
### Methods for securing data
Organisations combine multiple methods to preserve confidentiality, integrity, and availability.

- Geographic restrictions: limit access by region using IP intelligence and policy enforcement. This reduces exposure to high-risk locations and supports residency requirements.
- Encryption: converts data into unreadable form without the correct key. Symmetric encryption uses one shared key and is efficient for bulk data. Asymmetric encryption uses a public and private key pair and supports secure key exchange and identity. Encryption is commonly applied at disk, file, database, and network levels.
- Hashing: a one-way transformation used for integrity and verification. Password storage typically relies on specialised password hashing schemes rather than general-purpose hashes. Legacy algorithms such as MD5 are widely considered unsuitable for security-critical uses.
- Data masking: replaces or transforms sensitive fields so realistic datasets can be used safely for development, testing, and analytics. Common approaches include substitution and shuffling.
- Tokenisation: replaces sensitive values with tokens and stores the original values in a protected token vault. Tokens reduce exposure during processing and are common in payment and identity workflows.
- Obfuscation: makes code or data harder to interpret, which can protect intellectual property and hinder reverse engineering. It is not a substitute for encryption when confidentiality is required.
- Segmentation: separates systems, networks, or datasets so compromise is contained. Segmentation can be implemented through database partitioning, network zoning, and distinct access policies for different data categories.
- Permission restrictions: enforce least privilege and separation of duties so users and services can access only what they need. Common approaches include role-based access control, attribute-based access control, and multi-factor authentication.
### Database user profiles, password policies, privileges, and roles
Effective user management reduces the risk of unauthorised access and limits the impact of compromised credentials.

- User profiles should be unique per individual or service to support accountability. Profiles commonly define status, expiry, lockout thresholds, and resource limits.
- Password policy typically addresses length, complexity, rotation, reuse prevention, and the use of multi-factor authentication. Policies should be reviewed as threats and guidance evolve.
- Privileges define what actions a user can perform. System privileges affect the whole database, while object privileges apply to specific tables, views, and procedures.
- Roles group privileges so access aligns with job functions. Role hierarchies can reflect organisational structure, while regular reviews support least privilege and timely revocation.
### Designing database application security models
Database application security models translate business requirements and regulations into enforceable controls.

Core principles include confidentiality, integrity, availability, authentication, authorisation, and auditing and monitoring.

Common design steps:
- Identify security requirements by classifying data, mapping regulatory obligations, and assessing threats.
- Define policies for access control, encryption, backup and recovery, and incident response.
- Implement controls such as least privilege, segregation of duties, secure configuration, patching, encryption at rest and in transit, and centralised monitoring.
- Align practices with recognised standards, including ISO/IEC 27001, NIST SP 800-53, and CIS Controls.

Access control models used in databases commonly include discretionary access control, mandatory access control, and role-based access control. Encryption controls often include strong algorithms such as AES-256, robust key management, and database features such as transparent data encryption where appropriate.
### Database and application auditing
Auditing records activity so organisations can detect misuse, investigate incidents, and demonstrate compliance.

Key auditing objectives:
- Accountability and transparency through reliable activity records
- Compliance with legal and regulatory requirements
- Forensic readiness through detailed logs
- Operational insight into usage and performance

Designing an audit model typically includes:
- Defining what to audit, such as logins, privilege changes, schema changes, and sensitive data access
- Setting granularity, including user identifiers, timestamps, source addresses, and the affected objects
- Defining retention, protection, and access to audit records
- Selecting storage, such as database audit tables, external log files, or central log platforms

Implementation approaches include enabling DBMS audit features, using triggers for high-value events, and integrating with log and monitoring platforms such as SIEM solutions. Best practices include consistent timestamps, complete coverage, restricted access, encryption for logs in transit and at rest, tamper-evident controls, automated alerts, and scheduled reviews.

Application-level auditing complements database auditing by capturing application context and business events. Common approaches include embedding logging at key code paths, using middleware for centralised capture, and correlating application logs with database logs.
### Case study themes from major breaches, 2019 to 2023
Several high-profile breaches illustrate recurring weaknesses.

- Capital One, 2019: cloud configuration and access control weaknesses enabled unauthorised access to customer data.
- EasyJet, 2020: customer personal data exposure highlighted the need for strong application and identity controls.
- T-Mobile, 2021: an exposed API allowed access to customer identifiers, reinforcing the need for API hardening and monitoring.
- LastPass, 2022: compromise of development systems and theft of proprietary information emphasised software supply chain security and key management discipline.
- MOVEit, 2023: exploitation of a file transfer vulnerability demonstrated the impact of software flaws and the value of rapid patching, least privilege, and data minimisation.

Across cases, consistent lessons include strong cloud governance, secure software development, API security, third-party risk management, rapid vulnerability response, and clear incident communication.
### Data types and security implications
Different data types drive different controls and priorities.

- Trade secrets: protect with encryption, strong authentication, continuous monitoring, and restricted access.
- Intellectual property: protect with role-based access, encryption, patch management, and controlled distribution.
- Legal information: protect with confidentiality controls, integrity checks, secure backups, and defensible audit trails.
- Financial information: protect with strict access control, encryption, monitoring for fraud, and regular testing.
- Human-readable data: protect documents and exports with access control, encryption, and secure handling to reduce leakage.
- Machine-readable data: protect binaries and logs against tampering, ensure integrity, and restrict access to sensitive telemetry.
### Practical takeaways
- Classification and context determine the control set, not the storage location alone.
- Layered protections work best, including encryption, least privilege, segmentation, auditing, and resilient backup and recovery.
- Continuous monitoring, regular reviews, and timely patching reduce exposure as threats and systems change.
## Injection Vulnerabilities
### Injection flaws overview
- Injection flaws occur when an application passes attacker controlled data to an interpreter or query engine that treats the data as executable syntax.
- Typical targets include operating systems, SQL databases, NoSQL engines, directory services such as LDAP, and XML query processors.
- These weaknesses are commonly linked to unsafe input handling, dynamic command construction, and excessive privileges in the component that runs the command or query.
- Injection remains a high impact class of vulnerability and appears in major risk lists such as the OWASP Top 10 and the SANS Top 25.
### OS command injection
- OS command injection abuses application features that call operating system commands, for example file management features that delete or move files.
- The root cause is unsafe command execution combined with missing or weak input validation and sanitisation.
- Outcomes can include denial of service, data loss, disclosure of secrets such as passwords and keys, and full host compromise that can be used to pivot into the wider network.
### Reducing the need for OS commands
- The safest approach is to avoid executing OS commands from application code where practical.
- Language and library APIs often provide file and system operations that reduce exposure compared with running a shell command.
- Replacing shell calls with scoped library functions narrows what can be abused and limits the impact of malicious input.
### Safer execution when OS commands are unavoidable
- Least privilege reduces blast radius by running the application under an account that has only the permissions required for its task.
- Avoiding shell interpreters such as sh, bash, cmd, or PowerShell limits the attack surface by removing shell specific metacharacters and expansion rules.
- Commands should be executed directly with fixed arguments, using APIs that pass arguments as an array rather than a single concatenated string.
- Executables should be referenced using explicit paths to prevent path based substitution attacks, and to reduce the likelihood of DLL or shared library hijacking on misconfigured systems.
- Inputs should not reach the execution point unchanged. Using internal identifiers and mapping tables can translate user selections into known safe file names or targets.
### Input validation guidance
- Blacklists that attempt to block bad characters and tokens are fragile and are routinely bypassed through alternate encodings and shell features.
- Whitelists are more robust. They define an allowed character set, format, and length for each input, and reject anything outside the specification.
- Validation should occur as close as possible to the trust boundary, and business rules should enforce that the user can only act on authorised resources.
### SQL injection
- SQL injection exploits unsafe construction of SQL statements by mixing user supplied values into query text.
- Attackers can bypass authentication checks, read sensitive data, modify or delete records, and in some environments trigger additional database capabilities that increase impact.
- Even when an application does not display query results, attackers can sometimes infer data through differences in responses or timing behaviour.
### Common SQL injection techniques
- Error based attacks rely on verbose database errors returned to the user to learn schema details and refine payloads.
- Union based attacks attempt to combine results from the intended query with results from another query to extract additional data.
- Blind attacks use behavioural clues rather than direct output. Boolean based variants change application behaviour based on true or false outcomes, while time based variants rely on deliberate response delays.
- Out of band attacks attempt to exfiltrate data through secondary channels, such as external network requests initiated by the database environment.
### Preventing SQL injection
- Prepared statements and parameterised queries separate SQL structure from data values, keeping query logic fixed and treating user input only as data.
- Prepared statements must be used correctly. User input should not be concatenated into the query at preparation time.
- Input validation and allow lists still matter, particularly for fields that influence identifiers, sorting, or filtering options where parameters are harder to bind.
- Error handling should avoid exposing database brand, schema names, or raw query messages to end users, while retaining detailed logs for engineers.
- Database accounts used by applications should follow least privilege, including read only accounts where business needs permit.
- Stored procedures can reduce exposure when they are parameterised and avoid dynamic SQL.
- ORM libraries can lower risk by reducing raw SQL, but custom query features can reintroduce vulnerabilities if used unsafely.
- Web application firewalls can provide an additional detection and blocking layer, but they do not replace secure coding.
### Database injection beyond SQL
- NoSQL injection can occur when user controlled data is embedded into query objects or expressions, including JavaScript capable query features in some products.
- XPath injection can occur when user input is interpolated into XPath expressions used for searching XML documents.
- LDAP injection can occur when user input is concatenated into LDAP filter strings, allowing attackers to change the logic of directory queries.
- Across these technologies, the recurring controls are strict allow listed validation, avoiding dynamic expression building, and least privilege for the identity used to query the data store.
### Operational practices
- Regular security reviews and testing during the software development lifecycle help detect injection risks early.
- Secure configuration and patching reduce exposure to known flaws in frameworks, database engines, and supporting components.
- Developer and administrator training improves awareness of injection patterns and safe API usage.
### Notes on real world incidents
- SQL injection has contributed to major data breaches and is frequently observed in public incident reports.
- The Equifax breach in 2017 is widely attributed to an Apache Struts vulnerability, not SQL injection, but it illustrates how web application weaknesses can enable large scale data exposure.