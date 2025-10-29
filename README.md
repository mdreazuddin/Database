# Database

**What is a Database?**
> A database is an organized collection of data that can be stored, managed, and retrieved efficiently using a Database Management System (DBMS).

**Major Types of Databases**
> There are many, but here are common serveral Databases for DevOps and software engineer.

- **Relational Database (RDBMS):** Data stored in tables rows & columns with relationships. Use SQL for queries, best for Applications needing structured data, strong consistency, and relationships (e.g., ERP, banking, web apps).
  *Examples:* MySQL, PostgreSQL, Oracle, MariaDB, MS SQL Server.

- **NoSQL Database:** Non-tabular, flexible schema, stores unstructured or semi-structured data. Best for Big data, real-time analytics, caching, or apps needing flexibility in data models. *Example:* MongoDB, Cassandra, CouchDB.

- **NewSQL Database:** Combines SQL + NoSQL scalability. Best for Large-scale apps needing SQL consistency and NoSQL scalability. *Example:* CockroachDB, Google Spanner.

**Advantages & Limitations**
> Advantages of Relational Database (SQL)
- Strong data consistency (ACID properties).
- Supports complex queries & joins.
- Mature ecosystem, easy backup and replication.
- Great for structured data and transactions.

> Limitations of Relational Database (SQL)
- Scaling is hard (vertical scaling mostly).
- Fixed schema — hard to adapt to changing data.
- Performance drops for very large datasets.

> Advantages of NoSQL Database
- Flexible schema — easy to handle dynamic data.
- Highly scalable (horizontal scaling).
- Better performance for large unstructured data.
- Great for real-time web apps.

> Limitations of NoSQL Database
- No standard query language.
- Weaker consistency (eventual consistency).
- Complex transactions can be difficult.
