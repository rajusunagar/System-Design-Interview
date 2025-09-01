# Caching

## Definition
Caching is a technique to store frequently accessed data in a fast storage layer to reduce latency and improve performance.

## Cache Hierarchy

### Browser Cache
- **Location**: Client browser
- **Scope**: Single user
- **TTL**: Hours to days
- **Control**: HTTP headers (Cache-Control, Expires)

### CDN (Content Delivery Network)
- **Location**: Edge servers globally
- **Scope**: Geographic regions
- **Content**: Static assets, images, videos
- **Examples**: CloudFlare, AWS CloudFront

### Reverse Proxy Cache
- **Location**: Between client and server
- **Examples**: Nginx, Varnish
- **Use Case**: Cache dynamic content

### Application Cache
- **Location**: Application server memory
- **Examples**: Redis, Memcached
- **Scope**: Application-specific data

### Database Cache
- **Location**: Database server
- **Examples**: MySQL Query Cache, PostgreSQL shared_buffers
- **Scope**: Query results and data pages

## Caching Patterns

### Cache-Aside (Lazy Loading)
```
1. Check cache for data
2. If miss, fetch from database
3. Store in cache
4. Return data
```
- **Pros**: Only cache requested data
- **Cons**: Cache miss penalty

### Write-Through
```
1. Write to cache
2. Write to database
3. Return success
```
- **Pros**: Data consistency
- **Cons**: Higher write latency

### Write-Behind (Write-Back)
```
1. Write to cache
2. Return success immediately
3. Write to database asynchronously
```
- **Pros**: Low write latency
- **Cons**: Risk of data loss

### Refresh-Ahead
```
1. Proactively refresh cache before expiration
2. Based on access patterns
```
- **Pros**: Reduced cache misses
- **Cons**: Complex implementation

## Cache Eviction Policies

### LRU (Least Recently Used)
- Remove least recently accessed items
- **Good for**: Temporal locality patterns

### LFU (Least Frequently Used)
- Remove least frequently accessed items
- **Good for**: Frequency-based access patterns

### FIFO (First In, First Out)
- Remove oldest items first
- **Simple**: Easy to implement

### TTL (Time To Live)
- Remove items after expiration time
- **Predictable**: Time-based eviction

### Random
- Remove random items
- **Simple**: Low overhead

## Cache Levels

### L1 Cache (CPU)
- **Size**: 32KB - 64KB
- **Speed**: 1-2 cycles
- **Scope**: Per CPU core

### L2 Cache (CPU)
- **Size**: 256KB - 1MB
- **Speed**: 10-20 cycles
- **Scope**: Per CPU core or shared

### L3 Cache (CPU)
- **Size**: 8MB - 32MB
- **Speed**: 40-75 cycles
- **Scope**: Shared across cores

### RAM
- **Size**: GBs
- **Speed**: 100-300 cycles
- **Scope**: System-wide

### SSD/Disk
- **Size**: TBs
- **Speed**: Millions of cycles
- **Scope**: Persistent storage

## Distributed Caching

### Consistent Hashing
```
Hash(key) → Node
Handles node addition/removal gracefully
```

### Replication
- **Master-Slave**: One writer, multiple readers
- **Master-Master**: Multiple writers (conflict resolution needed)

### Partitioning
- **Range-based**: Partition by key ranges
- **Hash-based**: Partition by hash function

## Cache Invalidation

### TTL-based
- Set expiration time for cache entries
- **Pros**: Simple, automatic cleanup
- **Cons**: May serve stale data

### Event-based
- Invalidate cache on data changes
- **Pros**: Fresh data
- **Cons**: Complex implementation

### Manual
- Explicit cache invalidation
- **Pros**: Full control
- **Cons**: Error-prone

## Cache Warming Strategies

### Preloading
- Load cache with expected data before traffic
- **Use Case**: Application startup

### Gradual Warming
- Slowly increase cache hit ratio
- **Use Case**: New cache deployment

### Predictive Loading
- Load cache based on access patterns
- **Use Case**: Machine learning predictions

## Common Caching Technologies

### In-Memory
- **Redis**: Data structures, persistence, clustering
- **Memcached**: Simple key-value, high performance
- **Hazelcast**: Distributed computing platform

### Application-Level
- **Caffeine**: High-performance Java caching
- **Guava Cache**: Google's Java caching library
- **EHCache**: Java distributed caching

### HTTP Caching
- **Varnish**: HTTP accelerator
- **Squid**: Web proxy cache
- **Nginx**: Web server with caching

## Cache Metrics

### Hit Ratio
```
Hit Ratio = Cache Hits / (Cache Hits + Cache Misses)
Target: > 90% for most applications
```

### Latency
- **Cache Hit**: < 1ms
- **Cache Miss**: Database query time + cache write time

### Throughput
- Operations per second (OPS)
- **Redis**: 100K+ OPS
- **Memcached**: 1M+ OPS

### Memory Usage
- Monitor cache size vs. available memory
- Set appropriate eviction policies

## Best Practices

1. **Cache Hot Data**: Focus on frequently accessed data
2. **Set Appropriate TTL**: Balance freshness vs. performance
3. **Monitor Hit Ratios**: Aim for >90% hit ratio
4. **Handle Cache Failures**: Graceful degradation
5. **Avoid Cache Stampede**: Use locks or refresh-ahead
6. **Size Appropriately**: 80% of working set in cache

## Anti-Patterns

### Cache Everything
- **Problem**: Memory waste, poor hit ratios
- **Solution**: Cache selectively based on access patterns

### Infinite TTL
- **Problem**: Stale data, memory leaks
- **Solution**: Set appropriate expiration times

### Cache Stampede
- **Problem**: Multiple requests fetch same data simultaneously
- **Solution**: Use locks or single-flight pattern

## Interview Questions

1. Design a caching strategy for a social media feed
2. How do you handle cache invalidation in a distributed system?
3. What's the difference between Redis and Memcached?
4. How do you prevent cache stampede?
5. Design a multi-level caching system for an e-commerce site

## Real-World Examples

### Facebook
- **TAO**: Distributed data store with caching
- **Memcached**: Massive distributed cache
- **CDN**: Global content delivery

### Twitter
- **Redis**: Timeline caching
- **Manhattan**: Distributed key-value store
- **CDN**: Media content caching

### Netflix
- **EVCache**: Distributed in-memory cache
- **CDN**: Video content delivery
- **Zuul**: API gateway caching