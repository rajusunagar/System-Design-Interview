# SQL vs NoSQL

## Overview

### SQL Databases (RDBMS)
Relational databases that use Structured Query Language (SQL) for defining and manipulating data. Data is stored in tables with predefined schemas.

### NoSQL Databases
Non-relational databases designed for specific data models and have flexible schemas for building modern applications.

## Detailed Comparison

### Data Model

#### SQL Databases
```sql
-- Structured data with relationships
CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100) UNIQUE,
    created_at TIMESTAMP
);

CREATE TABLE posts (
    id INT PRIMARY KEY,
    user_id INT,
    title VARCHAR(200),
    content TEXT,
    created_at TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

#### NoSQL Databases
```javascript
// Document-based (MongoDB)
{
  "_id": "507f1f77bcf86cd799439011",
  "name": "John Doe",
  "email": "john@example.com",
  "posts": [
    {
      "title": "My First Post",
      "content": "Hello World!",
      "created_at": "2024-01-01T12:00:00Z"
    }
  ],
  "created_at": "2024-01-01T10:00:00Z"
}
```

### Schema Design

#### SQL - Fixed Schema
```sql
-- Schema must be defined upfront
ALTER TABLE users ADD COLUMN phone VARCHAR(20);
-- Requires migration for existing data
```

#### NoSQL - Flexible Schema
```javascript
// Can add fields dynamically
{
  "name": "Jane Doe",
  "email": "jane@example.com",
  "phone": "+1234567890",  // New field added
  "preferences": {         // Nested object
    "theme": "dark",
    "notifications": true
  }
}
```

### Query Language

#### SQL - Standardized
```sql
-- Complex joins and aggregations
SELECT u.name, COUNT(p.id) as post_count
FROM users u
LEFT JOIN posts p ON u.id = p.user_id
WHERE u.created_at > '2024-01-01'
GROUP BY u.id, u.name
HAVING COUNT(p.id) > 5
ORDER BY post_count DESC;
```

#### NoSQL - Varied Approaches
```javascript
// MongoDB aggregation pipeline
db.users.aggregate([
  {
    $match: {
      created_at: { $gt: new Date('2024-01-01') }
    }
  },
  {
    $lookup: {
      from: "posts",
      localField: "_id",
      foreignField: "user_id",
      as: "posts"
    }
  },
  {
    $project: {
      name: 1,
      post_count: { $size: "$posts" }
    }
  },
  {
    $match: {
      post_count: { $gt: 5 }
    }
  },
  {
    $sort: { post_count: -1 }
  }
]);
```

## NoSQL Database Types

### 1. Document Stores

#### Examples: MongoDB, CouchDB, Amazon DocumentDB

#### Use Cases
- Content management systems
- User profiles and preferences
- Product catalogs
- Real-time analytics

#### Data Structure
```javascript
// User profile document
{
  "_id": "user123",
  "profile": {
    "name": "John Doe",
    "age": 30,
    "location": {
      "city": "San Francisco",
      "country": "USA"
    }
  },
  "preferences": {
    "theme": "dark",
    "language": "en",
    "notifications": {
      "email": true,
      "push": false
    }
  },
  "activity": [
    {
      "action": "login",
      "timestamp": "2024-01-01T10:00:00Z",
      "ip": "192.168.1.1"
    }
  ]
}
```

#### Pros & Cons
**Pros:**
- Flexible schema
- Natural object mapping
- Rich query capabilities
- Horizontal scaling

**Cons:**
- No ACID transactions across documents
- Potential data duplication
- Complex queries can be inefficient

### 2. Key-Value Stores

#### Examples: Redis, DynamoDB, Riak

#### Use Cases
- Caching
- Session storage
- Shopping carts
- Real-time recommendations

#### Data Structure
```python
# Simple key-value pairs
user_session = {
    "session:abc123": {
        "user_id": "user456",
        "login_time": "2024-01-01T10:00:00Z",
        "last_activity": "2024-01-01T10:30:00Z"
    }
}

# Shopping cart
cart = {
    "cart:user456": {
        "items": [
            {"product_id": "prod123", "quantity": 2, "price": 29.99},
            {"product_id": "prod456", "quantity": 1, "price": 49.99}
        ],
        "total": 109.97
    }
}
```

#### Pros & Cons
**Pros:**
- Extremely fast reads/writes
- Simple data model
- Excellent for caching
- Linear scalability

**Cons:**
- Limited query capabilities
- No complex relationships
- Application must handle data structure

### 3. Column-Family (Wide Column)

#### Examples: Cassandra, HBase, Amazon Timestream

#### Use Cases
- Time-series data
- IoT sensor data
- Logging and analytics
- Content management

#### Data Structure
```
Row Key: user123
Column Family: profile
  name: "John Doe"
  email: "john@example.com"
  created_at: "2024-01-01"

Column Family: activity
  2024-01-01:10:00: "login"
  2024-01-01:10:15: "view_product:123"
  2024-01-01:10:30: "add_to_cart:456"
```

#### Pros & Cons
**Pros:**
- Excellent for time-series data
- High write throughput
- Flexible column structure
- Good compression

**Cons:**
- Complex data modeling
- Limited query flexibility
- Eventual consistency

### 4. Graph Databases

#### Examples: Neo4j, Amazon Neptune, ArangoDB

#### Use Cases
- Social networks
- Recommendation engines
- Fraud detection
- Knowledge graphs

#### Data Structure
```cypher
// Nodes and relationships
CREATE (john:Person {name: "John Doe", age: 30})
CREATE (jane:Person {name: "Jane Smith", age: 28})
CREATE (company:Company {name: "Tech Corp"})
CREATE (product:Product {name: "Smartphone", price: 699})

CREATE (john)-[:FRIENDS_WITH {since: "2020-01-01"}]->(jane)
CREATE (john)-[:WORKS_FOR {position: "Engineer"}]->(company)
CREATE (john)-[:PURCHASED {date: "2024-01-01"}]->(product)
CREATE (jane)-[:LIKES]->(product)
```

#### Pros & Cons
**Pros:**
- Natural representation of relationships
- Efficient traversal of connections
- Powerful query language (Cypher)
- Real-time insights

**Cons:**
- Specialized use cases
- Complex for simple data
- Scaling challenges

## When to Choose SQL vs NoSQL

### Choose SQL When:

#### 1. ACID Compliance Required
```sql
-- Financial transactions requiring consistency
BEGIN TRANSACTION;
UPDATE accounts SET balance = balance - 100 WHERE id = 'account1';
UPDATE accounts SET balance = balance + 100 WHERE id = 'account2';
COMMIT;
```

#### 2. Complex Queries and Reporting
```sql
-- Business intelligence queries
SELECT 
    DATE_TRUNC('month', order_date) as month,
    category,
    SUM(amount) as revenue,
    COUNT(*) as order_count,
    AVG(amount) as avg_order_value
FROM orders o
JOIN products p ON o.product_id = p.id
WHERE order_date >= '2024-01-01'
GROUP BY month, category
ORDER BY month, revenue DESC;
```

#### 3. Structured Data with Relationships
```sql
-- E-commerce with clear relationships
SELECT 
    c.name as customer_name,
    o.order_date,
    p.name as product_name,
    oi.quantity,
    oi.price
FROM customers c
JOIN orders o ON c.id = o.customer_id
JOIN order_items oi ON o.id = oi.order_id
JOIN products p ON oi.product_id = p.id
WHERE c.id = 123;
```

### Choose NoSQL When:

#### 1. Rapid Development and Iteration
```javascript
// Flexible schema for evolving requirements
const userProfile = {
    name: "John Doe",
    email: "john@example.com",
    // Easy to add new fields
    socialMedia: {
        twitter: "@johndoe",
        linkedin: "john-doe-123"
    },
    // Nested arrays and objects
    skills: ["JavaScript", "Python", "React"],
    experience: [
        {
            company: "Tech Corp",
            position: "Senior Developer",
            duration: "2020-2024"
        }
    ]
};
```

#### 2. Horizontal Scaling Requirements
```javascript
// Distributed across multiple nodes
const shardingStrategy = {
    shard1: { users: "user_id % 3 == 0" },
    shard2: { users: "user_id % 3 == 1" },
    shard3: { users: "user_id % 3 == 2" }
};
```

#### 3. Unstructured or Semi-structured Data
```javascript
// Varying document structures
const blogPost = {
    title: "My Blog Post",
    content: "...",
    tags: ["tech", "programming"],
    metadata: {
        readTime: "5 min",
        difficulty: "intermediate"
    }
};

const videoPost = {
    title: "Tutorial Video",
    videoUrl: "https://...",
    duration: 600,
    chapters: [
        { title: "Introduction", start: 0 },
        { title: "Main Content", start: 60 }
    ]
};
```

## Hybrid Approaches

### Polyglot Persistence
```python
class UserService:
    def __init__(self):
        self.postgres = PostgreSQLConnection()  # User profiles
        self.redis = RedisConnection()          # Sessions, cache
        self.mongodb = MongoDBConnection()      # User preferences
        self.neo4j = Neo4jConnection()         # Social graph
    
    def get_user_data(self, user_id):
        # Get structured data from PostgreSQL
        user_profile = self.postgres.query(
            "SELECT * FROM users WHERE id = %s", user_id
        )
        
        # Get session from Redis
        session = self.redis.get(f"session:{user_id}")
        
        # Get preferences from MongoDB
        preferences = self.mongodb.find_one(
            {"user_id": user_id}
        )
        
        # Get social connections from Neo4j
        friends = self.neo4j.query(
            "MATCH (u:User {id: $user_id})-[:FRIENDS_WITH]->(f) RETURN f",
            user_id=user_id
        )
        
        return {
            "profile": user_profile,
            "session": session,
            "preferences": preferences,
            "friends": friends
        }
```

### SQL with JSON Support
```sql
-- PostgreSQL JSON columns
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(200),
    price DECIMAL(10,2),
    attributes JSONB  -- Flexible attributes
);

INSERT INTO products (name, price, attributes) VALUES
('Laptop', 999.99, '{"brand": "TechCorp", "specs": {"ram": "16GB", "storage": "512GB SSD"}}'),
('Phone', 699.99, '{"brand": "PhoneCorp", "features": ["5G", "Wireless Charging"]}');

-- Query JSON data
SELECT name, price, attributes->>'brand' as brand
FROM products
WHERE attributes->'specs'->>'ram' = '16GB';
```

## Performance Considerations

### SQL Database Optimization
```sql
-- Indexing strategies
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_posts_user_date ON posts(user_id, created_at);
CREATE INDEX idx_orders_status_date ON orders(status, order_date) 
WHERE status IN ('pending', 'processing');

-- Query optimization
EXPLAIN ANALYZE
SELECT u.name, COUNT(p.id) as post_count
FROM users u
LEFT JOIN posts p ON u.id = p.user_id
WHERE u.created_at > '2024-01-01'
GROUP BY u.id, u.name;
```

### NoSQL Database Optimization
```javascript
// MongoDB indexing
db.users.createIndex({ "email": 1 }, { unique: true });
db.posts.createIndex({ "user_id": 1, "created_at": -1 });
db.posts.createIndex({ "tags": 1 });  // Array index

// Query optimization with explain
db.posts.find({ "user_id": "user123" }).explain("executionStats");

// Aggregation pipeline optimization
db.posts.aggregate([
    { $match: { "created_at": { $gte: new Date("2024-01-01") } } },
    { $group: { _id: "$user_id", count: { $sum: 1 } } },
    { $sort: { count: -1 } }
]).explain("executionStats");
```

## Migration Strategies

### SQL to NoSQL Migration
```python
class SQLToNoSQLMigrator:
    def __init__(self):
        self.mysql = MySQLConnection()
        self.mongodb = MongoDBConnection()
    
    def migrate_users(self):
        # Extract from SQL
        users = self.mysql.query("SELECT * FROM users")
        
        for user in users:
            # Transform to document format
            user_doc = {
                "_id": user['id'],
                "profile": {
                    "name": user['name'],
                    "email": user['email'],
                    "created_at": user['created_at']
                },
                "posts": self.get_user_posts(user['id']),
                "preferences": self.get_user_preferences(user['id'])
            }
            
            # Load into MongoDB
            self.mongodb.users.insert_one(user_doc)
    
    def get_user_posts(self, user_id):
        posts = self.mysql.query(
            "SELECT * FROM posts WHERE user_id = %s", user_id
        )
        return [
            {
                "title": post['title'],
                "content": post['content'],
                "created_at": post['created_at']
            }
            for post in posts
        ]
```

### Gradual Migration with Dual Writes
```python
class DualWriteService:
    def __init__(self):
        self.mysql = MySQLConnection()
        self.mongodb = MongoDBConnection()
        self.migration_percentage = 0.1  # 10% of reads from MongoDB
    
    def create_user(self, user_data):
        # Write to both systems
        mysql_id = self.mysql.insert_user(user_data)
        
        mongo_doc = self.transform_to_document(user_data, mysql_id)
        self.mongodb.users.insert_one(mongo_doc)
        
        return mysql_id
    
    def get_user(self, user_id):
        # Gradually shift reads to MongoDB
        if random.random() < self.migration_percentage:
            try:
                return self.mongodb.users.find_one({"_id": user_id})
            except Exception:
                # Fallback to MySQL
                return self.mysql.get_user(user_id)
        else:
            return self.mysql.get_user(user_id)
```

## Best Practices

### SQL Best Practices
1. **Normalize data** to reduce redundancy
2. **Use appropriate indexes** for query performance
3. **Implement connection pooling** for scalability
4. **Use prepared statements** to prevent SQL injection
5. **Monitor query performance** and optimize slow queries

### NoSQL Best Practices
1. **Design for your queries** - model data based on access patterns
2. **Denormalize when appropriate** for read performance
3. **Use appropriate consistency levels** based on requirements
4. **Implement proper error handling** for eventual consistency
5. **Monitor performance metrics** specific to your NoSQL database

## Interview Questions

1. **When to choose**: When would you choose SQL vs NoSQL for a new project?
2. **ACID vs BASE**: Explain the trade-offs between ACID and BASE properties
3. **Scaling**: How do SQL and NoSQL databases scale differently?
4. **Consistency**: How do you handle data consistency in NoSQL systems?
5. **Migration**: How would you migrate from SQL to NoSQL?

## Real-World Examples

### Netflix
- **Cassandra**: Viewing history, user preferences
- **MySQL**: Billing, user accounts
- **Elasticsearch**: Search and recommendations

### Facebook
- **MySQL**: User profiles, posts (heavily sharded)
- **Cassandra**: Messages, chat history
- **Graph databases**: Social connections

### Uber
- **MySQL**: Core business data, transactions
- **Cassandra**: Trip data, location history
- **Redis**: Real-time data, caching

The choice between SQL and NoSQL depends on your specific requirements for consistency, scalability, query complexity, and development speed. Many modern applications use a combination of both to leverage the strengths of each approach.