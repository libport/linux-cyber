# Database Essentials and Vulnerabilities
> [!NOTE]
> Database systems organise, retrieve, protect, and recover operational data. Sound database design combines an appropriate data model with controlled access, secure application code, reliable auditing, and tested recovery procedures.
## Database fundamentals
### Relational databases
A relational database stores data in tables. Each row represents a record, and each column represents an attribute. Keys and constraints define relationships and enforce rules that the database can check.

- A primary key uniquely identifies each row. It can contain one column or several columns.
- A foreign key links rows by referencing a primary key, a unique constraint, or another candidate key that the database product permits.
- A schema defines tables, columns, data types, constraints, views, and other objects.
- Normalisation can reduce unnecessary duplication and update anomalies. Deliberate denormalisation can improve some read-heavy workloads when testing justifies the trade-off.
- SQL supports data definition, querying, modification, access control, and transaction management, although each product implements its own dialect and extensions.
- ACID transactions provide atomicity, consistency, isolation, and durability. Consistency means that a committed transaction preserves declared database rules. It does not guarantee that application logic reflects every business rule.

Relational database management systems include IBM Db2, Microsoft SQL Server, MySQL, Oracle Database, and PostgreSQL. Organisations can run them on local devices, on-premises servers, virtual machines, containers, or managed database services.

Managed services often provide automated backups, replicas, monitoring, and failover. Replication improves availability, but it does not replace backups because it can copy accidental deletion, corruption, or malicious changes to replicas.
### NoSQL databases
NoSQL describes a broad group of non-relational database systems. The term is commonly interpreted as "not only SQL", but it does not define one architecture or consistency model.

Common NoSQL models include:
- Key-value stores, which retrieve values by key
- Document databases, which store structured documents such as JSON-like records
- Wide-column databases, which organise sparse data into column families
- Graph databases, which represent entities as nodes and relationships as edges

Many NoSQL products support flexible records and horizontal distribution, which can suit high-volume services and rapidly evolving data. These features are not universal. Some products enforce schemas, support multi-record transactions, or provide SQL-compatible query languages. Product selection should therefore compare query patterns, consistency requirements, transaction boundaries, latency, scale, operational skills, and recovery needs.

Relational systems usually suit workloads that depend on strong constraints, multi-table transactions, and flexible joins. NoSQL systems can suit workloads built around aggregates, graph traversal, sparse records, or very large distributed datasets. Neither category guarantees better performance or security.
### Entity-relationship modelling
An entity-relationship model describes entities, their attributes, and the relationships between them. In a relational implementation, entities commonly become tables, and attributes become columns.

Typical column types include fixed-length and variable-length text, integers, exact decimals, binary values, dates, times, and timestamps. Designers should choose the narrowest type that accurately represents the data. For example, financial amounts normally require an exact decimal type rather than binary floating point.

Keys express identity and relationships:
- A candidate key can uniquely identify a row.
- A primary key is the candidate key selected as the main row identifier.
- An alternate key is a candidate key not selected as the primary key.
- A foreign key requires a referenced value to exist unless the foreign-key column permits `NULL`.
- A composite key contains more than one column.

Database constraints should enforce rules that remain true for every write, including `NOT NULL`, `UNIQUE`, `CHECK`, primary-key, and foreign-key constraints. Application validation can improve usability, but it should not replace database integrity controls.
### Car dealership schema
A basic dealership schema can use three tables:

| Table | Primary key | Selected columns | Relationships |
| --- | --- | --- | --- |
| `car` | `serial_no` | `model`, `manufacturer`, `price` | A sale can reference a car |
| `salesperson` | `salesperson_id` | `given_name`, `family_name` | A sale can reference a salesperson |
| `sale` | `sale_id` | `serial_no`, `salesperson_id`, `sale_date`, `sale_price` | Foreign keys reference `car` and `salesperson` |

The `sale` table needs its own identifier because a sale is an independent business event. If the business permits each car to be sold only once, a `UNIQUE` constraint on `sale.serial_no` can enforce that rule. If the business records resale events, the schema should allow several sales for the same car and distinguish them with `sale_id`.
## SQL basics and database management
### SELECT queries and result sets
A `SELECT` statement retrieves rows and returns a result set. A projection chooses columns, while a `WHERE` clause filters rows with a predicate. SQL predicates can evaluate to true, false, or unknown because `NULL` represents a missing or inapplicable value.

```sql
SELECT book_id, title
FROM book
WHERE book_id = 'B1'
```

Character literals normally use single quotes. Identifiers follow product-specific quoting and case rules.

Common comparison operators include `=`, `<`, `>`, `<=`, `>=`, and `<>`. A comparison with `NULL` does not return true, so SQL uses predicates such as `IS NULL` and `IS NOT NULL`.
### Aggregation, distinct values, and row limits
`COUNT(*)` counts rows. `COUNT(column_name)` counts rows in which the named column is not `NULL`. `DISTINCT` removes duplicate result rows after the query evaluates the selected expressions.

```sql
SELECT COUNT(*)
FROM medal
```

```sql
SELECT DISTINCT country
FROM medal
WHERE medal_type = 'GOLD'
```

MySQL, PostgreSQL, and SQLite support `LIMIT`. Other products use forms such as `TOP` or `FETCH FIRST`. A row limit without `ORDER BY` does not define which qualifying rows the database will return.

```sql
SELECT country, medal_type
FROM medal
WHERE competition_year = 2018
ORDER BY country, medal_type
LIMIT 5
```
### Inserting, updating, and deleting data
`INSERT` adds rows. Listing target columns makes the statement clearer and protects it from some schema changes.

```sql
INSERT INTO author
    (author_id, family_name, given_name, email, city, country_code)
VALUES
    ('A1', 'Chong', 'Raul', 'rfc@ibm.com', 'Toronto', 'CA'),
    ('A2', 'Ahuja', 'Rav', 'rav@ibm.com', 'Bengaluru', 'IN')
```

`UPDATE` changes existing rows, and `DELETE` removes rows. Without a qualifying `WHERE` clause, either statement can affect every row that the account can modify.

```sql
UPDATE author
SET family_name = 'Katta', given_name = 'Lakshmi'
WHERE author_id = 'A2'
```

```sql
DELETE FROM author
WHERE author_id IN ('A2', 'A3')
```

Applications should run related changes in a transaction, check the affected row count, and handle failures explicitly. Database permissions should prevent an application account from changing tables that its business function does not require.
### SQL command categories
SQL terminology varies across standards, products, and training material. Common categories include:
- Data definition language, or DDL, for schema objects through commands such as `CREATE`, `ALTER`, and `DROP`
- Data manipulation language, or DML, for reading and changing rows through commands such as `SELECT`, `INSERT`, `UPDATE`, and `DELETE`
- Data control language, or DCL, for privileges through commands such as `GRANT` and `REVOKE`
- Transaction control for operations such as `COMMIT`, `ROLLBACK`, and `SAVEPOINT`

Some sources classify `SELECT` as data query language, and products differ in how they classify or implement commands such as `TRUNCATE`. Product documentation should govern administration and recovery decisions.
### Roles, privileges, and least privilege
Privileges authorise operations at scopes such as the server, database, schema, table, column, row, function, or procedure. Available scopes and names vary by product.

Roles group privileges around job functions or service responsibilities. Administrators should grant access to roles, assign roles to identities, and review both regularly. Effective privileges do not always equal a simple union because explicit denials, role activation, ownership, inheritance, and row-level policies can change the result.

Application services should use separate identities for distinct trust levels. Read-only functions should use accounts that cannot write. Migration and administration accounts should not serve routine application traffic.
### Backups and recovery
Backups support recovery from hardware failure, operator error, corruption, ransomware, and other incidents. A sound strategy starts with a recovery point objective and a recovery time objective, then selects methods that can meet both.

| Method | Primary use | Important limitation |
| --- | --- | --- |
| Logical export | Migration, selective recovery, and cross-version movement | Restore can take substantial time and usually requires a working target engine |
| Physical or base backup | Full instance or cluster recovery | Files, logs, versions, and storage consistency must align with product requirements |
| Full backup | Complete recovery baseline | Large datasets can require significant time and storage |
| Differential backup | Changes since a designated full backup in products that support it | Restore depends on the correct full backup and product-specific backup chain |
| Incremental or log backup | Changes since an earlier backup or log position | Definitions and restore sequences differ by product |
| Continuous log archiving | Point-in-time recovery when combined with a valid base backup | Missing or damaged log segments can break recovery |

Organisations should automate backups, encrypt and restrict them, retain independent copies, monitor every job, and test restoration. Recovery tests should verify data integrity, application compatibility, credentials, keys, dependencies, and elapsed recovery time.

Replicas and failover systems support continuity, but they can reproduce destructive changes. Snapshots can also share failure domains or administrative credentials with production. Neither approach removes the need for protected, versioned backups and tested restoration.
## Database security and data protection
### Data classification
Data classification assigns handling requirements according to sensitivity, legal obligations, business value, and operational impact. No universal hierarchy defines labels such as confidential, private, restricted, or critical. Each organisation should publish clear definitions and map them to controls.

A practical handling hierarchy can use these levels:

| Level | Meaning | Typical controls |
| --- | --- | --- |
| Public | Approved for public release | Integrity checks, change control, and availability protection |
| Internal | Intended for authorised personnel and routine business use | Authentication, access control, and approved sharing channels |
| Confidential | Disclosure could harm individuals or the organisation | Need-to-know access, encryption, monitoring, and contractual controls |
| Restricted | Disclosure or alteration could cause severe harm or breach a legal duty | Strong authentication, tightly limited access, detailed auditing, encryption, and enhanced recovery controls |

Separate tags can identify personal information, health information, payment data, government identifiers, legal material, trade secrets, and other regulated or high-value data. A criticality rating can then express availability and integrity needs. These dimensions prevent a single label from obscuring different risks.

Data owners should approve classifications, retention periods, access rules, and disposal methods. Systems should carry the classification through exports, backups, logs, test datasets, and downstream services.
### Data at rest, in transit, and in use
Controls change with the state of the data:
- Data at rest resides in database files, indexes, exports, snapshots, and backups. Encryption, key management, access control, auditing, and secure disposal reduce risk.
- Data in transit crosses networks or service boundaries. Properly configured TLS can provide encryption, peer authentication, and integrity. A bare unkeyed hash does not prevent an attacker from changing both the data and its hash.
- Data in use appears in memory, processor registers, caches, and application workflows. Secure coding, process isolation, access control, runtime protection, and minimised exposure reduce risk. Confidential-computing technologies can address selected threats but do not replace application controls.

Encryption at rest protects storage media and copied files, but it does not stop an authorised database session from reading plaintext. Transparent data encryption therefore complements, rather than replaces, access control and application-layer protection.
### Data location, residency, and sovereignty
Data residency identifies where an organisation stores or processes data. Data sovereignty concerns the laws and regulatory powers that apply. Location can influence jurisdiction, but obligations can also follow the organisation, the affected people, the data type, the recipient, and the transfer arrangement.

Australian entities covered by the Privacy Act must assess Australian Privacy Principle requirements when they disclose personal information overseas. Contract terms, technical controls, provider locations, support access, backups, and subprocessors all affect cross-border risk.

IP geolocation can support risk-based access decisions, but it can be inaccurate and can be bypassed through relays, virtual private networks, or compromised devices. It does not prove where a provider stores data or establish compliance with residency requirements.
### Protection methods
Layered controls protect confidentiality, integrity, and availability:
- Encryption transforms plaintext with a cryptographic algorithm and key. Symmetric encryption suits bulk data. Public-key cryptography supports functions such as key establishment and digital signatures. Secure key generation, storage, rotation, revocation, separation, and recovery determine whether either approach remains effective.
- Hashing maps input to a fixed-length digest. Unkeyed cryptographic hashes can detect accidental changes when a trusted digest exists. Message authentication codes or digital signatures provide stronger protection against malicious alteration.
- Password hashing uses a salted, adaptive function designed to resist offline guessing, such as Argon2id, scrypt, bcrypt, or an appropriately configured PBKDF2 implementation. Fast general-purpose hashes such as MD5 or SHA-256 do not provide suitable password storage on their own.
- Masking transforms sensitive fields for lower-trust uses such as development or analytics. Static test data should not retain a reversible path to production values unless the use case requires and protects that path.
- Tokenisation replaces a sensitive value with a token. Some designs store the original in a protected vault, while others use different controlled mappings. The security assessment must cover the token service, mapping data, access paths, and re-identification risk.
- Obfuscation makes code or data harder to understand. It can delay analysis, but it does not provide cryptographic confidentiality.
- Segmentation separates trust zones and restricts communication. Network zones, separate database instances, separate accounts, row-level security, and controlled service interfaces can create meaningful boundaries. Table partitioning alone does not create a security boundary.
- Data minimisation limits collection, retention, replication, and exposure. Data that the organisation no longer holds cannot be stolen from its systems.
### Identities, credentials, privileges, and roles
Each person and service should use a unique identity. Shared administrator or application accounts weaken accountability and complicate revocation.

Identity controls should cover:
- Provisioning, approval, suspension, expiry, and prompt removal
- Strong authentication and multi-factor authentication for privileged or high-risk access
- Service-account ownership, credential rotation, and non-interactive use
- Login throttling, compromise detection, and secure recovery
- Privileged access management for exceptional administrative work

Password policy should favour length, block common or compromised passwords, permit password managers, and avoid arbitrary composition rules that encourage weak patterns. Organisations should not force periodic password changes without evidence of compromise. They should require a change when compromise occurs and should protect stored verifiers with an approved password-hashing scheme.

Privileges should align with job functions and application operations. Roles simplify assignment, while attribute-based and row-level policies can refine decisions using context. Administrators should test effective access because ownership, inherited roles, explicit denials, and product defaults can produce unexpected permissions.
### Security architecture
A database security architecture should start with data flows, trust boundaries, threat scenarios, legal obligations, and recovery objectives. It should then define controls for authentication, authorisation, confidentiality, integrity, availability, auditing, and incident response.

Core design decisions include:
- Isolation between internet-facing services and database networks
- Encrypted and authenticated connections
- Separate identities and databases for development, testing, and production
- Least privilege for applications, analysts, support staff, and administrators
- Secure storage and rotation of database credentials and encryption keys
- Patch, configuration, dependency, and vulnerability management
- Protected backups and rehearsed recovery
- Central monitoring and documented incident procedures

Frameworks can organise the control program. Relevant current baselines include ISO/IEC 27001:2022, NIST SP 800-53 Revision 5, and CIS Controls Version 8.1. Organisations should tailor controls to their risks rather than treating a framework as a universal checklist.
### Database and application auditing
Auditing records security-relevant actions for detection, investigation, accountability, compliance, and operational analysis. Native database audit facilities usually capture events more reliably than application code alone. Application logs add business context that the database cannot see, such as the user journey, transaction purpose, and authorisation decision.

An audit design should define:
- Events such as authentication, failed authorisation, privilege changes, schema changes, sensitive reads, bulk exports, and administrative actions
- Attributes such as a stable actor identifier, timestamp, source, target object, action, result, and correlation identifier
- Retention, access, encryption, integrity protection, disposal, and legal constraints
- Alert thresholds, review responsibilities, escalation paths, and incident links

Triggers can record selected high-value changes, but they can add overhead, miss operations outside their scope, and depend on secure administration. Native audit features and external collection usually provide a stronger foundation.

Logging systems should synchronise time, restrict access, detect interruption or tampering, and send important events to a separate security domain. Logs should not record passwords, access tokens, encryption keys, full connection strings, or unnecessary personal information.
### Lessons from major incidents, 2019 to 2023
| Incident | Established facts | Security lesson |
| --- | --- | --- |
| Capital One, 2019 | An intruder obtained personal information relating to about 106 million people. The US banking regulator later cited ineffective cloud risk assessment and delayed remediation of deficiencies. | Cloud controls require tested risk assessment, effective internal audit, least privilege, and prompt remediation. |
| easyJet, 2020 | The airline reported access to email addresses and booking details for about nine million bookers and payment-card details for 2,208 customers. Public disclosures did not establish a detailed technical root cause. | Data minimisation, monitoring, response readiness, and clear notification reduce harm when the root cause remains under investigation. |
| T-Mobile, 2021 | The company reported that an attacker reached testing environments, then used brute-force attacks and other methods to access servers containing customer data. | Testing environments need production-grade identity controls, segmentation, monitoring, and data restrictions. |
| LastPass, 2022 | Two linked incidents exposed development information and later enabled access to cloud backups containing secrets, customer metadata, and encrypted and unencrypted customer data. | Organisations must protect developer endpoints, secrets, privileged access, backup environments, and encryption boundaries as one connected system. |
| T-Mobile, 2023 | The company reported unauthorised retrieval of data for about 37 million customer accounts through one API. This event was separate from the 2021 intrusion. | API authorisation, data minimisation, rate controls, monitoring, and rapid containment limit large-scale extraction. |
| MOVEit Transfer, 2023 | Attackers exploited CVE-2023-34362, a SQL injection vulnerability in the web application, to access affected MOVEit databases. | Internet-facing software needs secure query construction, rapid vulnerability response, exposure reduction, and post-compromise data controls. |
## Injection vulnerabilities
### Injection flaws
Injection occurs when software sends attacker-controlled data to an interpreter that treats part of the data as executable syntax. Targets include SQL and NoSQL query engines, operating-system commands, LDAP filters, XPath expressions, and template or expression languages.

Injection usually combines unsafe dynamic construction with inadequate separation between code and data. Excessive privileges increase the damage after exploitation. The OWASP Top 10:2025 includes injection as A05, and the 2025 CWE Top 25 ranks SQL injection and OS command injection among its highest-risk weaknesses.
### OS command injection
OS command injection occurs when an application incorporates untrusted input into a system command. Successful exploitation can disclose or destroy data, steal credentials, disrupt services, execute arbitrary code, or provide a path into connected systems.

Applications should avoid OS commands when a language or library API can perform the operation. A scoped file API, for example, exposes less interpreter behaviour than a shell command.

When an OS command remains necessary, the application should:
- Invoke the executable directly without a shell interpreter
- Pass arguments as a structured array rather than a concatenated command string
- Fix the executable and permitted operation in code or trusted configuration
- Map user choices to internal identifiers and approved values
- Validate each argument against its expected type, length, range, and character set
- Use an explicit executable path, a controlled working directory, a minimal environment, and secure dependency-loading settings
- Run under an identity with only the required operating-system and file permissions
- Record execution attempts, failures, and unusual frequency without logging secrets

An explicit executable path reduces path-search substitution, but it does not by itself prevent malicious library loading or unsafe behaviour inside the called program.
### Input validation
Allow-list validation defines accepted types, formats, lengths, ranges, and values. Deny lists that search for dangerous characters or strings are easy to bypass through alternative syntax, encoding, or interpreter features.

Validation should occur when data crosses a trust boundary and again where a component needs a stricter contract. Canonicalisation should precede security decisions when several encodings can represent the same value. Validation does not replace parameterisation or safe execution APIs because valid business data can still contain interpreter metacharacters.

Authorisation must also bind each request to an allowed object. A syntactically valid file name, customer identifier, or record key does not prove that the requester can use it.
### SQL injection
SQL injection occurs when software combines untrusted values with SQL text in a way that changes the intended statement structure. Attackers can bypass checks, read sensitive rows, change or delete data, and sometimes invoke dangerous database features.

Common techniques include:
- Error-based injection, which uses detailed database errors to refine an attack
- Union-based injection, which appends compatible query results to exposed output
- Boolean-based blind injection, which infers information from differences between true and false responses
- Time-based blind injection, which infers information from controlled delays
- Out-of-band injection, which uses a secondary channel when the database and network configuration permit it
### Preventing SQL injection
Prepared statements and parameterised queries provide the primary defence for data values. They keep the SQL structure fixed and send values separately. Server-side parameterisation is essential because a client library that concatenates strings before transmission does not provide the same protection.

Applications should apply these controls together:
- Parameterise every value that the driver and database permit
- Use allow-listed mappings for table names, column names, sort directions, and other structural elements that parameters cannot represent
- Avoid dynamic SQL where a fixed statement can express the operation
- Construct necessary dynamic SQL with product-supported identifier quoting and tightly controlled templates
- Treat stored procedures as safe only when they parameterise values and avoid unsafe dynamic SQL
- Review raw-query and escape-hatch features in object-relational mapping libraries
- Give application accounts only the required tables, views, procedures, rows, and operations
- Return generic errors to users while recording protected diagnostic details for authorised staff
- Test query-building code through review, static analysis, automated tests, and security testing
- Use a web application firewall as an additional detection or containment layer, not as the primary fix

Input validation supports these controls but cannot replace parameterisation. Escaping is database-specific and fragile, so applications should reserve it for cases in which safer designs cannot work and should use maintained product libraries.
### Injection beyond SQL
NoSQL injection can arise when applications accept untrusted query operators, JavaScript expressions, or loosely typed objects. Applications should build queries through safe APIs, enforce schemas and types, reject unexpected operators, disable unnecessary script execution, and apply least privilege.

LDAP injection can arise when applications concatenate input into directory filters or distinguished names. Context-specific encoding, safe directory APIs, allow-list validation, and restricted directory permissions reduce risk.

XPath injection can arise when applications interpolate input into expressions used to search XML. Parameterised APIs, fixed expressions, controlled mappings, and least privilege reduce exposure.

The same principle applies across interpreters: software should keep code fixed, pass data through a safe interface, validate it against the business contract, authorise the requested action, and limit the interpreter's privileges.
### Operational practices
Secure development and operations reinforce code-level controls:
- Threat modelling identifies interpreter boundaries and high-impact data paths.
- Code review focuses on string-built queries, shell calls, dynamic expressions, and privileged database features.
- Automated security tests cover parameters, headers, cookies, JSON fields, file metadata, and other inputs.
- Patch and configuration management reduce exposure in database engines, frameworks, drivers, and supporting software.
- Monitoring detects error spikes, unusual query patterns, repeated validation failures, bulk access, and unexpected outbound connections.
- Incident procedures preserve logs, revoke exposed credentials, isolate affected services, assess data access, and restore trusted operation.

The 2017 Equifax breach did not originate from SQL injection. Attackers exploited an unpatched Apache Struts vulnerability in an internet-facing application, then reached additional databases. The incident demonstrates how patching failures, weak segmentation, exposed credentials, insufficient monitoring, and permissive query activity can turn one application flaw into extensive data loss.