# Scalability

## Definition
Scalability is the ability of a system to handle increased load by adding resources to the system.

## Types of Scalability

### 1. Vertical Scaling (Scale Up)
- Adding more power (CPU, RAM) to existing machines
- **Pros**: Simple, no code changes needed
- **Cons**: Hardware limits, single point of failure, expensive

### 2. Horizontal Scaling (Scale Out)
- Adding more machines to the pool of resources
- **Pros**: No hardware limits, fault tolerant, cost-effective
- **Cons**: Complex, requires load distribution

## Scalability Patterns

### Load Distribution
```
Client → Load Balancer → [Server1, Server2, Server3]
```

### Database Scaling
- **Read Replicas**: Multiple read-only copies
- **Sharding**: Horizontal partitioning
- **Federation**: Split databases by function

### Caching Layers
- **Browser Cache**: Client-side caching
- **CDN**: Geographic distribution
- **Application Cache**: In-memory storage
- **Database Cache**: Query result caching

## Key Metrics
- **Throughput**: Requests per second (RPS)
- **Latency**: Response time
- **Concurrent Users**: Active users at once
- **Resource Utilization**: CPU, Memory, Disk, Network

## Scalability Bottlenecks
1. **Database**: Often the first bottleneck
2. **Network**: Bandwidth limitations
3. **CPU**: Processing power limits
4. **Memory**: RAM constraints
5. **Disk I/O**: Storage speed limits

## Best Practices
- Design for horizontal scaling from day one
- Use stateless services
- Implement proper caching strategies
- Monitor and measure performance
- Plan for peak load scenarios

## Real-World Examples
- **Netflix**: Microservices architecture
- **Amazon**: Service-oriented architecture
- **Google**: Distributed systems design
- **Facebook**: Horizontal scaling with sharding

## Interview Questions
1. How would you scale a web application from 1K to 1M users?
2. What are the trade-offs between vertical and horizontal scaling?
3. How do you identify scalability bottlenecks?
4. Design a system that can handle 10x traffic increase.