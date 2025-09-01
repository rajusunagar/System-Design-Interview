# Databases

## SQL vs NoSQL

### SQL Databases (RDBMS)
- **ACID Properties**: Atomicity, Consistency, Isolation, Durability
- **Schema**: Fixed schema, structured data
- **Relationships**: Foreign keys, joins
- **Examples**: MySQL, PostgreSQL, Oracle, SQL Server

### NoSQL Databases
- **Flexible Schema**: Dynamic, unstructured data
- **Horizontal Scaling**: Built for distributed systems
- **Types**: Document, Key-Value, Column-family, Graph
- **Examples**: MongoDB, Cassandra, Redis, Neo4j

## NoSQL Types

### Document Stores
- **Structure**: JSON-like documents
- **Use Cases**: Content management, catalogs, user profiles
- **Examples**: MongoDB, CouchDB, Amazon DocumentDB
```json
{
  "_id": "user123",
  "name": "John Doe",
  "email": "john@example.com",
  "preferences": {
    "theme": "dark",
    "notifications": true
  }
}
```

### Key-Value Stores
- **Structure**: Simple key-value pairs
- **Use Cases**: Caching, session storage, shopping carts
- **Examples**: Redis, DynamoDB, Riak
```
user:123 → {"name": "John", "email": "john@example.com"}
session:abc → {"userId": 123, "expires": 1640995200}
```

### Column-Family
- **Structure**: Column-oriented storage
- **Use Cases**: Time-series data, IoT, analytics
- **Examples**: Cassandra, HBase, Amazon Timestream
```
Row Key: user123
Columns: name:John, email:john@example.com, age:30
```

### Graph Databases
- **Structure**: Nodes and relationships
- **Use Cases**: Social networks, recommendation engines
- **Examples**: Neo4j, Amazon Neptune, ArangoDB
```
(User:John)-[:FOLLOWS]->(User:Jane)
(User:John)-[:LIKES]->(Post:123)
```

## Database Scaling Patterns

### Vertical Scaling (Scale Up)
- Add more CPU, RAM, storage to existing server
- **Pros**: Simple, no application changes
- **Cons**: Hardware limits, expensive, single point of failure

### Horizontal Scaling (Scale Out)

#### Read Replicas
```
Master (Write) → Replica 1 (Read)
              → Replica 2 (Read)
              → Replica 3 (Read)
```
- **Pros**: Scales read operations
- **Cons**: Eventual consistency, replication lag

#### Sharding (Horizontal Partitioning)
```python
def get_shard(user_id):
    return user_id % num_shards

# Shard 1: users 1, 4, 7, 10...
# Shard 2: users 2, 5, 8, 11...
# Shard 3: users 3, 6, 9, 12...
```

### Sharding Strategies

#### Range-based Sharding
```
Shard 1: user_id 1-1000000
Shard 2: user_id 1000001-2000000
Shard 3: user_id 2000001-3000000
```
- **Pros**: Simple range queries
- **Cons**: Hotspots, uneven distribution

#### Hash-based Sharding
```python
shard = hash(user_id) % num_shards
```
- **Pros**: Even distribution
- **Cons**: Range queries difficult

#### Directory-based Sharding
```
Lookup Service:
user123 → shard2
user456 → shard1
user789 → shard3
```
- **Pros**: Flexible, can rebalance
- **Cons**: Additional complexity, lookup overhead

### Federation (Functional Partitioning)
```
Users DB: user profiles, authentication
Posts DB: posts, comments, likes
Messages DB: chat messages, notifications
```
- **Pros**: Clear separation, independent scaling
- **Cons**: Cross-database joins difficult

## Replication Patterns

### Master-Slave Replication
```
Master (Read/Write) → Slave 1 (Read Only)
                   → Slave 2 (Read Only)
```
- **Consistency**: Eventual consistency
- **Failover**: Manual promotion of slave to master

### Master-Master Replication
```
Master 1 (Read/Write) ↔ Master 2 (Read/Write)
```
- **Availability**: No single point of failure
- **Complexity**: Conflict resolution needed

### Synchronous vs Asynchronous

#### Synchronous Replication
- Write confirmed only after all replicas updated
- **Pros**: Strong consistency
- **Cons**: Higher latency, availability issues

#### Asynchronous Replication
- Write confirmed immediately, replicas updated later
- **Pros**: Low latency, high availability
- **Cons**: Potential data loss, eventual consistency

## Consistency Models

### Strong Consistency
- All nodes see the same data simultaneously
- **Examples**: Traditional RDBMS, MongoDB (default)
- **Use Cases**: Financial systems, inventory

### Eventual Consistency
- System will become consistent over time
- **Examples**: DynamoDB, Cassandra
- **Use Cases**: Social media, content delivery

### Weak Consistency
- No guarantees about when data will be consistent
- **Examples**: Memcached, some caching systems
- **Use Cases**: Real-time gaming, live streaming

## Database Selection Criteria

### Choose SQL When:
- **ACID Requirements**: Financial transactions
- **Complex Queries**: Reporting, analytics
- **Structured Data**: Well-defined schema
- **Mature Ecosystem**: Existing tools and expertise

### Choose NoSQL When:
- **Horizontal Scaling**: Massive scale requirements
- **Flexible Schema**: Rapid development, changing requirements
- **High Performance**: Low latency, high throughput
- **Specific Use Cases**: Document storage, caching, graphs

## Popular Database Technologies

### SQL Databases

#### MySQL
- **Strengths**: Mature, widely adopted, good performance
- **Use Cases**: Web applications, e-commerce
- **Scaling**: Read replicas, sharding with tools like Vitess

#### PostgreSQL
- **Strengths**: Advanced features, JSON support, extensible
- **Use Cases**: Complex applications, geospatial data
- **Scaling**: Streaming replication, logical replication

#### Amazon RDS
- **Strengths**: Managed service, automated backups, scaling
- **Options**: MySQL, PostgreSQL, Oracle, SQL Server
- **Features**: Multi-AZ deployment, read replicas

### NoSQL Databases

#### MongoDB
- **Type**: Document store
- **Strengths**: Flexible schema, rich queries, aggregation
- **Scaling**: Sharding, replica sets
- **Use Cases**: Content management, real-time analytics

#### Cassandra
- **Type**: Column-family
- **Strengths**: Linear scalability, high availability
- **Scaling**: Automatic sharding, multi-datacenter
- **Use Cases**: Time-series data, IoT, messaging

#### Redis
- **Type**: Key-value store
- **Strengths**: In-memory performance, data structures
- **Features**: Pub/sub, transactions, Lua scripting
- **Use Cases**: Caching, session storage, real-time analytics

#### DynamoDB
- **Type**: Key-value/Document
- **Strengths**: Managed service, predictable performance
- **Scaling**: Automatic scaling, global tables
- **Use Cases**: Gaming, mobile apps, serverless applications

## Database Design Best Practices

### Schema Design
```sql
-- Good: Normalized design
CREATE TABLE users (
    user_id BIGINT PRIMARY KEY,
    username VARCHAR(50) UNIQUE,
    email VARCHAR(100) UNIQUE
);

CREATE TABLE posts (
    post_id BIGINT PRIMARY KEY,
    user_id BIGINT,
    content TEXT,
    created_at TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(user_id)
);
```

### Indexing Strategy
```sql
-- Primary key index (automatic)
-- Frequently queried columns
CREATE INDEX idx_posts_user_time ON posts(user_id, created_at);

-- Composite indexes for multi-column queries
CREATE INDEX idx_posts_status_time ON posts(status, created_at);

-- Partial indexes for filtered queries
CREATE INDEX idx_active_users ON users(created_at) WHERE status = 'active';
```

### Denormalization for Performance
```sql
-- Instead of joining every time
SELECT u.username, p.content 
FROM posts p 
JOIN users u ON p.user_id = u.user_id;

-- Store username in posts table
CREATE TABLE posts (
    post_id BIGINT PRIMARY KEY,
    user_id BIGINT,
    username VARCHAR(50), -- Denormalized
    content TEXT,
    created_at TIMESTAMP
);
```

## Performance Optimization

### Query Optimization
- **Use EXPLAIN**: Analyze query execution plans
- **Proper Indexing**: Cover frequently queried columns
- **Avoid N+1 Queries**: Use joins or batch queries
- **Limit Results**: Use LIMIT/OFFSET for pagination

### Connection Pooling
```python
# Connection pool configuration
pool = ConnectionPool(
    host='localhost',
    database='mydb',
    user='user',
    password='password',
    minconn=1,
    maxconn=20
)
```

### Caching Strategies
- **Query Result Caching**: Cache expensive query results
- **Object Caching**: Cache frequently accessed objects
- **Page Caching**: Cache entire web pages
- **Database Query Cache**: Built-in database caching

## Monitoring and Maintenance

### Key Metrics
- **Query Performance**: Slow query log, execution time
- **Connection Usage**: Active connections, connection pool
- **Resource Usage**: CPU, memory, disk I/O
- **Replication Lag**: Delay between master and replicas

### Backup Strategies
- **Full Backups**: Complete database snapshot
- **Incremental Backups**: Changes since last backup
- **Point-in-Time Recovery**: Restore to specific timestamp
- **Cross-Region Backups**: Disaster recovery

### Database Maintenance
- **Index Maintenance**: Rebuild fragmented indexes
- **Statistics Updates**: Keep query optimizer statistics current
- **Partition Maintenance**: Archive old data
- **Capacity Planning**: Monitor growth trends

## Interview Questions

1. **Design Decisions**: When would you choose SQL vs NoSQL?
2. **Scaling**: How do you scale a database to handle 10x more traffic?
3. **Consistency**: Explain the trade-offs between consistency and availability
4. **Sharding**: How do you handle cross-shard queries?
5. **Replication**: What are the pros and cons of master-master replication?

## Real-World Examples

### Facebook (Meta)
- **MySQL**: Heavily modified for scale
- **TAO**: Distributed data store for social graph
- **Memcached**: Massive caching layer

### Netflix
- **Cassandra**: Viewing history, recommendations
- **MySQL**: Billing, user accounts
- **EVCache**: Distributed caching

### Uber
- **MySQL**: Core business data
- **Cassandra**: Trip data, analytics
- **Redis**: Real-time data, caching