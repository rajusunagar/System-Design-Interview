# CAP Theorem

## Definition
CAP Theorem states that in a distributed system, you can only guarantee 2 out of 3 properties:

## The Three Properties

### Consistency (C)
- All nodes see the same data simultaneously
- Every read receives the most recent write
- **Strong Consistency**: Immediate consistency across all nodes
- **Eventual Consistency**: System will become consistent over time

### Availability (A)
- System remains operational 100% of the time
- Every request receives a response (success or failure)
- No downtime even during network failures

### Partition Tolerance (P)
- System continues to operate despite network failures
- Communication between nodes may fail
- Essential for distributed systems

## CAP Combinations

### CP Systems (Consistency + Partition Tolerance)
- **Examples**: MongoDB, Redis, HBase
- **Trade-off**: May become unavailable during network partitions
- **Use Case**: Financial systems, inventory management

### AP Systems (Availability + Partition Tolerance)
- **Examples**: Cassandra, DynamoDB, CouchDB
- **Trade-off**: May return stale data during partitions
- **Use Case**: Social media feeds, recommendation systems

### CA Systems (Consistency + Availability)
- **Examples**: Traditional RDBMS (MySQL, PostgreSQL)
- **Trade-off**: Cannot handle network partitions
- **Reality**: Not truly distributed, single point of failure

## Real-World Implications

### Banking System (CP)
```
Account Balance: $1000
Transfer: $500 to another account
Result: Either both accounts updated or transaction fails
Never: Inconsistent balances
```

### Social Media Feed (AP)
```
User posts update
Some users see it immediately
Others see it after few seconds
Acceptable: Temporary inconsistency
```

## PACELC Theorem Extension
**If Partition (P)**: Choose between Availability (A) and Consistency (C)
**Else (E)**: Choose between Latency (L) and Consistency (C)

### Examples:
- **PA/EL**: Cassandra, DynamoDB
- **PC/EC**: MongoDB, Redis
- **PA/EC**: PNUTS
- **PC/EL**: Rarely practical

## Design Decisions

### Choose CP When:
- Data accuracy is critical
- Financial transactions
- Inventory management
- User authentication

### Choose AP When:
- User experience is priority
- Social media platforms
- Content delivery
- Analytics systems

## Interview Questions
1. Explain CAP theorem with real-world examples
2. How does Netflix handle CAP theorem trade-offs?
3. Design a system that prioritizes availability over consistency
4. What happens to a CP system during network partition?
5. How do you achieve eventual consistency?

## Common Misconceptions
- **Myth**: You must choose only 2 properties
- **Reality**: You can have all 3 during normal operation
- **Truth**: During network partition, choose between C and A

## Practical Strategies
- **Multi-region deployment**: Reduce partition probability
- **Circuit breakers**: Handle partition gracefully
- **Eventual consistency**: Reconcile data after partition
- **Conflict resolution**: Handle concurrent updates