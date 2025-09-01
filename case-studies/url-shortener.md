# Design URL Shortener (like bit.ly)

## Problem Statement
Design a URL shortening service that converts long URLs into short URLs and redirects users when they access the short URL.

## Requirements

### Functional Requirements
1. **Shorten URL**: Given a long URL, return a short URL
2. **Redirect**: When accessing short URL, redirect to original URL
3. **Custom URLs**: Allow users to create custom short URLs
4. **Expiration**: URLs can have expiration dates
5. **Analytics**: Track click counts and basic analytics

### Non-Functional Requirements
1. **Scale**: 100M URLs shortened per day
2. **Read Heavy**: 100:1 read to write ratio
3. **Latency**: < 100ms for redirection
4. **Availability**: 99.9% uptime
5. **Durability**: URLs should not be lost

## Capacity Estimation

### Traffic
- **Write**: 100M URLs/day = 1,157 URLs/second
- **Read**: 10B redirects/day = 115,700 redirects/second
- **Peak**: 2x average = 231,400 redirects/second

### Storage
- **URL Entry**: 500 bytes (original URL + metadata)
- **Daily Storage**: 100M × 500 bytes = 50GB/day
- **5 Years**: 50GB × 365 × 5 = 91TB

### Bandwidth
- **Incoming**: 1,157 × 500 bytes = 578KB/s
- **Outgoing**: 115,700 × 500 bytes = 57MB/s

### Memory (Caching)
- **80-20 Rule**: 20% URLs generate 80% traffic
- **Cache Size**: 115,700 × 3600 × 24 × 0.2 = 2B requests/day
- **Memory**: 2B × 500 bytes = 1TB

## API Design

### Shorten URL
```http
POST /api/v1/shorten
{
  "longUrl": "https://example.com/very/long/url",
  "customAlias": "mylink", // optional
  "expiration": "2024-12-31T23:59:59Z" // optional
}

Response:
{
  "shortUrl": "https://short.ly/abc123",
  "longUrl": "https://example.com/very/long/url",
  "createdAt": "2024-01-01T00:00:00Z",
  "expiration": "2024-12-31T23:59:59Z"
}
```

### Redirect
```http
GET /abc123
Response: 302 Redirect to original URL
```

### Analytics
```http
GET /api/v1/analytics/abc123
Response:
{
  "shortUrl": "https://short.ly/abc123",
  "clicks": 1500,
  "clicksByDate": {...},
  "topCountries": {...}
}
```

## Database Design

### URL Mapping Table
```sql
CREATE TABLE urls (
    short_url VARCHAR(7) PRIMARY KEY,
    long_url TEXT NOT NULL,
    user_id BIGINT,
    created_at TIMESTAMP,
    expiration_date TIMESTAMP,
    click_count BIGINT DEFAULT 0,
    INDEX idx_user_id (user_id),
    INDEX idx_created_at (created_at)
);
```

### Analytics Table
```sql
CREATE TABLE url_analytics (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    short_url VARCHAR(7),
    clicked_at TIMESTAMP,
    ip_address VARCHAR(45),
    user_agent TEXT,
    country VARCHAR(2),
    FOREIGN KEY (short_url) REFERENCES urls(short_url),
    INDEX idx_short_url_time (short_url, clicked_at)
);
```

## URL Encoding Strategies

### Base62 Encoding
- **Characters**: [a-z, A-Z, 0-9] = 62 characters
- **Length 6**: 62^6 = 56B unique URLs
- **Length 7**: 62^7 = 3.5T unique URLs

### Counter-based Approach
```python
def encode_base62(num):
    chars = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789"
    result = ""
    while num > 0:
        result = chars[num % 62] + result
        num //= 62
    return result
```

### Hash-based Approach
```python
import hashlib

def generate_short_url(long_url):
    hash_value = hashlib.md5(long_url.encode()).hexdigest()
    return hash_value[:7]  # Take first 7 characters
```

### Random Generation
```python
import random
import string

def generate_random_key(length=7):
    chars = string.ascii_letters + string.digits
    return ''.join(random.choice(chars) for _ in range(length))
```

## High-Level Architecture

```
[Client] → [Load Balancer] → [Web Servers] → [Cache] → [Database]
                                ↓
                           [Analytics Service]
                                ↓
                           [Analytics DB]
```

### Components

#### Web Servers
- Handle URL shortening and redirection
- Stateless for horizontal scaling
- Rate limiting per user/IP

#### Cache Layer
- **Redis/Memcached**: Cache popular URLs
- **TTL**: 24 hours for URL mappings
- **Eviction**: LRU policy

#### Database
- **Primary**: MySQL/PostgreSQL for URL mappings
- **Sharding**: By hash of short URL
- **Replication**: Master-slave for read scaling

#### Analytics Service
- Asynchronous processing of click events
- Message queue (Kafka/RabbitMQ)
- Batch processing for aggregations

## Detailed Design

### URL Shortening Flow
```
1. Client sends long URL
2. Check if URL already exists in cache/DB
3. If exists, return existing short URL
4. Generate new short URL
5. Store mapping in database
6. Cache the mapping
7. Return short URL to client
```

### Redirection Flow
```
1. Client requests short URL
2. Check cache for mapping
3. If cache miss, query database
4. If found, cache the mapping
5. Redirect to original URL
6. Log analytics event asynchronously
```

### Handling Duplicates
- **Hash long URL**: Check if already shortened
- **User-specific**: Same URL can have different short URLs per user
- **Deduplication**: Option to return existing short URL

## Scaling Strategies

### Database Sharding
```python
def get_shard(short_url):
    return hash(short_url) % num_shards
```

### Caching Strategy
- **Cache hot URLs**: 80-20 rule
- **Multi-level caching**: L1 (local), L2 (Redis)
- **Cache warming**: Preload popular URLs

### Read Replicas
- **Master**: Handle writes (URL creation)
- **Slaves**: Handle reads (redirections)
- **Load balancing**: Route reads to replicas

### CDN Integration
- Cache popular redirections at edge
- Reduce latency for global users
- Handle static assets (analytics dashboard)

## Security Considerations

### Rate Limiting
```python
# Per user: 100 URLs/hour
# Per IP: 1000 requests/hour
```

### Malicious URLs
- **Blacklist**: Known malicious domains
- **Scanning**: Integrate with security services
- **Reporting**: Allow users to report spam

### Custom URLs
- **Reserved words**: Prevent system conflicts
- **Profanity filter**: Block inappropriate content
- **Length limits**: Minimum 3, maximum 20 characters

## Monitoring & Analytics

### Key Metrics
- **URL creation rate**: URLs/second
- **Redirection rate**: Redirects/second
- **Cache hit ratio**: >90% target
- **Response time**: <100ms P99
- **Error rate**: <0.1%

### Analytics Features
- **Click tracking**: Count and timestamps
- **Geographic data**: Country/city analysis
- **Referrer tracking**: Traffic sources
- **Device analytics**: Mobile vs desktop

## Advanced Features

### Bulk Operations
```http
POST /api/v1/bulk-shorten
{
  "urls": ["url1", "url2", "url3"]
}
```

### QR Code Generation
- Generate QR codes for short URLs
- Cache generated QR codes
- Different sizes and formats

### API Keys & Authentication
- User accounts and API keys
- Rate limiting per API key
- Usage analytics per user

## Interview Discussion Points

1. **Encoding Strategy**: Compare different approaches
2. **Database Choice**: SQL vs NoSQL trade-offs
3. **Caching Strategy**: Multi-level caching design
4. **Scaling**: Horizontal scaling approaches
5. **Analytics**: Real-time vs batch processing
6. **Security**: Handling malicious content
7. **Global Scale**: Multi-region deployment

## Follow-up Questions

1. How would you handle 10x more traffic?
2. How to implement real-time analytics?
3. How to prevent abuse and spam?
4. How to handle database failures?
5. How to implement A/B testing for different algorithms?

## Real-World Examples

### bit.ly
- Uses MongoDB for storage
- Redis for caching
- Real-time analytics
- Custom domains support

### TinyURL
- Simple hash-based approach
- No user accounts
- Minimal analytics
- High availability focus

### Google URL Shortener (goo.gl)
- Integrated with Google services
- Advanced analytics
- Security scanning
- Deprecated in 2019