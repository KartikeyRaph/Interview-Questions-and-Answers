# Databases Interview Q&A

---

## SQL Databases

### Q1: What is normalization? Why is it important?

**Answer:**
Normalization is the process of organizing data in a database to reduce redundancy and improve data integrity. It involves dividing tables into related tables and defining relationships. Common normal forms:
- 1NF: Eliminate repeating groups
- 2NF: Remove partial dependencies
- 3NF: Remove transitive dependencies

**Benefits:**
- Reduces data duplication
- Improves consistency
- Simplifies updates and deletes

---

### Q2: What are the differences between SQL and NoSQL databases?

**Answer:**
- **SQL:** Relational, structured schema, ACID compliance, tables/rows (e.g., PostgreSQL, MySQL, SQL Server)
- **NoSQL:** Non-relational, flexible schema, eventual consistency, documents/keys/graphs (e.g., MongoDB, Redis, Elasticsearch)

**Use Cases:**
- SQL: Complex queries, transactions, structured data
- NoSQL: Scalability, unstructured data, fast key-value access

---

### Q3: What are ACID properties?

**Answer:**
- **Atomicity:** All operations in a transaction succeed or none do
- **Consistency:** Database remains in a valid state
- **Isolation:** Transactions do not affect each other
- **Durability:** Changes persist after commit

---

### Q4: How do you optimize SQL queries?

**Answer:**
- Use indexes on frequently queried columns
- Avoid SELECT *; specify columns
- Use JOINs efficiently
- Analyze query plans (EXPLAIN)
- Avoid subqueries when possible
- Use proper data types
- Partition large tables

---

### Q5: PostgreSQL: How do you use window functions?

**Answer:**
Window functions perform calculations across rows related to the current row.

```sql
SELECT employee_id, salary,
       RANK() OVER (ORDER BY salary DESC) AS salary_rank,
       AVG(salary) OVER (PARTITION BY department_id) AS avg_dept_salary
FROM employees;
```

**Common window functions:** RANK(), ROW_NUMBER(), LEAD(), LAG(), SUM(), AVG()

---

### Q6: MySQL: How do you handle replication and high availability?

**Answer:**
- **Replication:** Master-slave (asynchronous), master-master (multi-source)
- **High Availability:** Use MySQL Group Replication, InnoDB Cluster, ProxySQL
- **Failover:** Use tools like MHA, Orchestrator

**Example:**
```sql
-- Enable binary logging for replication
[mysqld]
log-bin=mysql-bin
server-id=1
```

**Key Points:**
- Monitor replication lag
- Use GTIDs for easier failover
- Regularly backup and test restores

---

### Q7: SQL Server: What are stored procedures and how do you use them?

**Answer:**
Stored procedures are precompiled SQL code that can be executed repeatedly.

**Benefits:**
- Encapsulate business logic
- Improve performance
- Enhance security

**Example:**
```sql
CREATE PROCEDURE GetEmployeeById
    @EmployeeId INT
AS
BEGIN
    SELECT * FROM Employees WHERE EmployeeId = @EmployeeId
END
```

**Usage:**
```sql
EXEC GetEmployeeById @EmployeeId = 101;
```

---

### Q11: Scenario: How do you troubleshoot a slow-running SQL query?

**Answer:**
- Use `EXPLAIN` or `EXPLAIN ANALYZE` to view the query plan
- Check for missing indexes on WHERE/JOIN columns
- Look for full table scans
- Optimize joins and subqueries
- Check server resources (CPU, memory, disk I/O)
- Rewrite query for efficiency (avoid correlated subqueries)
- Partition large tables if needed

---

### Q12: Scenario: How do you handle a deadlock in a transactional system?

**Answer:**
- Identify deadlock using database logs or monitoring tools
- Use shorter transactions and commit early
- Access tables in a consistent order
- Use appropriate isolation levels (e.g., READ COMMITTED)
- Retry transactions automatically if deadlock detected

---

### Q13: Scenario: How do you migrate a large production database with minimal downtime?

**Answer:**
- Use replication to sync data to new server
- Perform cutover during off-peak hours
- Use tools like pg_dump/pg_restore (PostgreSQL), mysqldump (MySQL), SQL Server Management Studio
- Test migration on staging environment
- Monitor application and rollback if issues occur

---

## NoSQL Databases

### Q8: MongoDB: How do you model relationships in MongoDB?

**Answer:**
- **Embedded documents:** Store related data inside a parent document (one-to-few)
- **References:** Store ObjectId references to other collections (one-to-many, many-to-many)

**Example:**
```json
// Embedded
{
  "_id": 1,
  "name": "John",
  "addresses": [
    {"city": "NYC", "zip": "10001"},
    {"city": "LA", "zip": "90001"}
  ]
}

// Reference
{
  "_id": 2,
  "name": "Jane",
  "address_ids": [ObjectId("..."), ObjectId("...")]
}
```

**Key Points:**
- Use embedding for performance, referencing for flexibility
- Use `$lookup` for joins in aggregation pipeline

---

### Q9: Redis: What are common use cases and data structures?

**Answer:**
- **Use Cases:** Caching, session storage, pub/sub, rate limiting, leaderboards
- **Data Structures:** Strings, Lists, Sets, Hashes, Sorted Sets, Streams

**Example:**
```bash
# Set and get a value
SET user:1:name "Alice"
GET user:1:name

# List operations
LPUSH recent_users "Bob"
LRANGE recent_users 0 10

# Hash operations
HSET user:1 profile "admin"
HGET user:1 profile
```

**Key Points:**
- In-memory, extremely fast
- Supports persistence (RDB, AOF)
- Expiry and eviction policies for cache

---

### Q10: Elasticsearch: How do you design efficient search queries?

**Answer:**
- Use mappings to define field types (text, keyword, date, etc.)
- Use analyzers for full-text search
- Use filters for exact matches, queries for scoring
- Paginate results with `from` and `size`
- Use aggregations for analytics

**Example:**
```json
GET /products/_search
{
  "query": {
    "bool": {
      "must": [
        {"match": {"name": "laptop"}}
      ],
      "filter": [
        {"term": {"brand": "Dell"}}
      ]
    }
  },
  "aggs": {
    "avg_price": {"avg": {"field": "price"}}
  }
}
```

**Key Points:**
- Use bulk indexing for performance
- Monitor cluster health and shard allocation
- Use Kibana for visualization

---

### Q14: Scenario: How do you design a schema for a multi-tenant SaaS application?

**Answer:**
- Use a `tenant_id` column in all tables (shared schema)
- Separate databases per tenant (isolated schema)
- Use partitioning/sharding for scalability
- Secure data access with row-level security
- Index tenant_id for fast lookups

---

### Q15: Scenario: How do you ensure data consistency in a distributed NoSQL system?

**Answer:**
- Use write and read quorums (e.g., MongoDB, Cassandra)
- Enable replication and configure consistency levels
- Use distributed transactions if supported
- Monitor replication lag and resolve conflicts
- Design for eventual consistency if strict consistency is not required

---

### Q16: Scenario: How do you implement full-text search in a product catalog?

**Answer:**
- Use Elasticsearch or PostgreSQL full-text search
- Define analyzers and mappings for product fields
- Index product name, description, tags
- Use fuzzy matching and relevance scoring
- Paginate and aggregate results for analytics

---

### Q17: Scenario: How do you cache frequently accessed data in a high-traffic application?

**Answer:**
- Use Redis or Memcached for in-memory caching
- Cache query results, session data, computed values
- Set appropriate TTL (time-to-live) for cache keys
- Invalidate cache on data updates
- Monitor cache hit/miss rates

---

## Best Practices for Databases

- Always use parameterized queries to prevent SQL injection
- Regularly backup and test restores
- Monitor performance and slow queries
- Use connection pooling
- Choose the right database for your use case
- Document schema and data flows
- Secure access with roles and permissions
- Scale horizontally (sharding, replication) when needed

---

**Last Updated:** November 11, 2025
