# Rate Limiting

## Definition
Rate limiting is a technique to control the rate of requests sent or received by a system, preventing abuse and ensuring fair usage of resources.

## Why Rate Limiting?

### Protection Against
- **DDoS Attacks**: Overwhelming the system with requests
- **Brute Force Attacks**: Password guessing, API key enumeration
- **Resource Exhaustion**: CPU, memory, database connections
- **Cost Control**: Limiting expensive operations (API calls, SMS)

### Benefits
- **System Stability**: Prevent overload and crashes
- **Fair Usage**: Ensure all users get fair access
- **Revenue Protection**: Prevent abuse of paid services
- **SLA Compliance**: Maintain service level agreements

## Rate Limiting Algorithms

### 1. Token Bucket Algorithm

#### Concept
- Bucket holds tokens at a fixed capacity
- Tokens are added at a constant rate
- Each request consumes one token
- If no tokens available, request is rejected

#### Implementation
```python
import time
import threading

class TokenBucket:
    def __init__(self, capacity, refill_rate):
        self.capacity = capacity
        self.tokens = capacity
        self.refill_rate = refill_rate  # tokens per second
        self.last_refill = time.time()
        self.lock = threading.Lock()
    
    def consume(self, tokens=1):
        with self.lock:
            self._refill()
            if self.tokens >= tokens:
                self.tokens -= tokens
                return True
            return False
    
    def _refill(self):
        now = time.time()
        tokens_to_add = (now - self.last_refill) * self.refill_rate
        self.tokens = min(self.capacity, self.tokens + tokens_to_add)
        self.last_refill = now

# Usage
bucket = TokenBucket(capacity=10, refill_rate=1)  # 10 tokens, 1 per second

def handle_request():
    if bucket.consume():
        return "Request processed"
    else:
        return "Rate limit exceeded", 429
```

#### Characteristics
- **Burst Handling**: Allows bursts up to bucket capacity
- **Smooth Rate**: Steady token refill rate
- **Memory**: O(1) per bucket

### 2. Leaky Bucket Algorithm

#### Concept
- Requests enter a bucket (queue)
- Requests are processed at a fixed rate
- If bucket overflows, requests are dropped

#### Implementation
```python
import time
import threading
from collections import deque

class LeakyBucket:
    def __init__(self, capacity, leak_rate):
        self.capacity = capacity
        self.leak_rate = leak_rate  # requests per second
        self.bucket = deque()
        self.last_leak = time.time()
        self.lock = threading.Lock()
    
    def add_request(self, request):
        with self.lock:
            self._leak()
            if len(self.bucket) < self.capacity:
                self.bucket.append(request)
                return True
            return False
    
    def _leak(self):
        now = time.time()
        elapsed = now - self.last_leak
        requests_to_leak = int(elapsed * self.leak_rate)
        
        for _ in range(min(requests_to_leak, len(self.bucket))):
            self.bucket.popleft()
        
        self.last_leak = now

# Usage
bucket = LeakyBucket(capacity=100, leak_rate=10)  # 100 capacity, 10/sec rate

def handle_request(request):
    if bucket.add_request(request):
        return "Request queued"
    else:
        return "Bucket full", 429
```

#### Characteristics
- **Smooth Output**: Constant processing rate
- **Queue Behavior**: Requests are queued, not immediately processed
- **Memory**: O(capacity)

### 3. Fixed Window Counter

#### Concept
- Count requests in fixed time windows
- Reset counter at window boundaries
- Allow requests if under limit

#### Implementation
```python
import time
import threading
from collections import defaultdict

class FixedWindowCounter:
    def __init__(self, limit, window_size):
        self.limit = limit
        self.window_size = window_size  # seconds
        self.counters = defaultdict(int)
        self.lock = threading.Lock()
    
    def is_allowed(self, key):
        with self.lock:
            current_window = int(time.time() // self.window_size)
            
            # Clean old windows
            self._cleanup_old_windows(current_window)
            
            if self.counters[f"{key}:{current_window}"] < self.limit:
                self.counters[f"{key}:{current_window}"] += 1
                return True
            return False
    
    def _cleanup_old_windows(self, current_window):
        keys_to_delete = []
        for key in self.counters:
            window = int(key.split(':')[1])
            if window < current_window:
                keys_to_delete.append(key)
        
        for key in keys_to_delete:
            del self.counters[key]

# Usage
limiter = FixedWindowCounter(limit=100, window_size=60)  # 100 requests per minute

def handle_request(user_id):
    if limiter.is_allowed(user_id):
        return "Request processed"
    else:
        return "Rate limit exceeded", 429
```

#### Characteristics
- **Simple**: Easy to understand and implement
- **Burst Issue**: Allows 2x limit at window boundaries
- **Memory**: O(active windows × users)

### 4. Sliding Window Log

#### Concept
- Keep log of request timestamps
- Count requests in sliding time window
- Remove old requests from log

#### Implementation
```python
import time
import threading
from collections import defaultdict, deque

class SlidingWindowLog:
    def __init__(self, limit, window_size):
        self.limit = limit
        self.window_size = window_size
        self.logs = defaultdict(deque)
        self.lock = threading.Lock()
    
    def is_allowed(self, key):
        with self.lock:
            now = time.time()
            cutoff = now - self.window_size
            
            # Remove old requests
            while self.logs[key] and self.logs[key][0] <= cutoff:
                self.logs[key].popleft()
            
            if len(self.logs[key]) < self.limit:
                self.logs[key].append(now)
                return True
            return False

# Usage
limiter = SlidingWindowLog(limit=100, window_size=3600)  # 100 requests per hour

def handle_request(user_id):
    if limiter.is_allowed(user_id):
        return "Request processed"
    else:
        return "Rate limit exceeded", 429
```

#### Characteristics
- **Accurate**: No burst issues
- **Memory Intensive**: Stores all request timestamps
- **Performance**: O(log size) per request

### 5. Sliding Window Counter

#### Concept
- Hybrid of fixed window and sliding window
- Estimate current window count using previous window
- More accurate than fixed window, less memory than sliding log

#### Implementation
```python
import time
import threading
from collections import defaultdict

class SlidingWindowCounter:
    def __init__(self, limit, window_size):
        self.limit = limit
        self.window_size = window_size
        self.counters = defaultdict(lambda: {'count': 0, 'timestamp': 0})
        self.lock = threading.Lock()
    
    def is_allowed(self, key):
        with self.lock:
            now = time.time()
            current_window = int(now // self.window_size)
            previous_window = current_window - 1
            
            # Get current and previous window counts
            current_key = f"{key}:{current_window}"
            previous_key = f"{key}:{previous_window}"
            
            current_count = self.counters[current_key]['count']
            previous_count = self.counters.get(previous_key, {}).get('count', 0)
            
            # Calculate sliding window estimate
            elapsed_in_current = now - (current_window * self.window_size)
            weight = 1 - (elapsed_in_current / self.window_size)
            estimated_count = (previous_count * weight) + current_count
            
            if estimated_count < self.limit:
                self.counters[current_key]['count'] += 1
                self.counters[current_key]['timestamp'] = now
                return True
            return False

# Usage
limiter = SlidingWindowCounter(limit=100, window_size=60)  # 100 requests per minute
```

#### Characteristics
- **Balanced**: Good accuracy with reasonable memory usage
- **Approximation**: Not perfectly accurate but close
- **Memory**: O(active windows × users)

## Distributed Rate Limiting

### Redis-based Implementation
```python
import redis
import time
import json

class RedisRateLimiter:
    def __init__(self, redis_client, limit, window_size):
        self.redis = redis_client
        self.limit = limit
        self.window_size = window_size
    
    def is_allowed(self, key):
        pipe = self.redis.pipeline()
        now = time.time()
        
        # Sliding window log approach with Redis
        cutoff = now - self.window_size
        
        # Remove old entries
        pipe.zremrangebyscore(key, 0, cutoff)
        
        # Count current entries
        pipe.zcard(key)
        
        # Add current request
        pipe.zadd(key, {str(now): now})
        
        # Set expiration
        pipe.expire(key, int(self.window_size) + 1)
        
        results = pipe.execute()
        current_count = results[1]
        
        return current_count < self.limit

# Usage with Redis Cluster
redis_client = redis.Redis(host='localhost', port=6379, db=0)
limiter = RedisRateLimiter(redis_client, limit=1000, window_size=3600)

def handle_request(user_id):
    if limiter.is_allowed(f"user:{user_id}"):
        return "Request processed"
    else:
        return "Rate limit exceeded", 429
```

### Lua Script for Atomic Operations
```lua
-- Redis Lua script for token bucket
local key = KEYS[1]
local capacity = tonumber(ARGV[1])
local tokens = tonumber(ARGV[2])
local interval = tonumber(ARGV[3])
local requested = tonumber(ARGV[4])

local bucket = redis.call('HMGET', key, 'tokens', 'last_refill')
local current_tokens = tonumber(bucket[1]) or capacity
local last_refill = tonumber(bucket[2]) or redis.call('TIME')[1]

local now = redis.call('TIME')[1]
local elapsed = now - last_refill
local new_tokens = math.min(capacity, current_tokens + (elapsed * tokens / interval))

if new_tokens >= requested then
    new_tokens = new_tokens - requested
    redis.call('HMSET', key, 'tokens', new_tokens, 'last_refill', now)
    redis.call('EXPIRE', key, interval * 2)
    return 1
else
    redis.call('HMSET', key, 'tokens', new_tokens, 'last_refill', now)
    redis.call('EXPIRE', key, interval * 2)
    return 0
end
```

```python
class RedisTokenBucket:
    def __init__(self, redis_client):
        self.redis = redis_client
        self.script = self.redis.register_script(LUA_SCRIPT)
    
    def consume(self, key, capacity, refill_rate, interval, tokens=1):
        result = self.script(
            keys=[key],
            args=[capacity, refill_rate, interval, tokens]
        )
        return bool(result)
```

## Rate Limiting Strategies

### Per-User Rate Limiting
```python
class UserRateLimiter:
    def __init__(self):
        self.limiters = {
            'free': TokenBucket(capacity=100, refill_rate=1),      # 100/hour
            'premium': TokenBucket(capacity=1000, refill_rate=10), # 1000/hour
            'enterprise': TokenBucket(capacity=10000, refill_rate=100) # 10000/hour
        }
    
    def is_allowed(self, user_id, user_tier):
        limiter = self.limiters.get(user_tier, self.limiters['free'])
        return limiter.consume()
```

### Per-IP Rate Limiting
```python
class IPRateLimiter:
    def __init__(self):
        self.limiter = SlidingWindowCounter(limit=1000, window_size=3600)
    
    def is_allowed(self, ip_address):
        return self.limiter.is_allowed(ip_address)
```

### Per-API Endpoint Rate Limiting
```python
class EndpointRateLimiter:
    def __init__(self):
        self.limiters = {
            '/api/search': TokenBucket(capacity=100, refill_rate=10),
            '/api/upload': TokenBucket(capacity=10, refill_rate=1),
            '/api/download': TokenBucket(capacity=50, refill_rate=5)
        }
    
    def is_allowed(self, endpoint, user_id):
        key = f"{endpoint}:{user_id}"
        limiter = self.limiters.get(endpoint)
        return limiter.consume() if limiter else True
```

## Advanced Rate Limiting Patterns

### Hierarchical Rate Limiting
```python
class HierarchicalRateLimiter:
    def __init__(self):
        self.global_limiter = TokenBucket(capacity=10000, refill_rate=100)
        self.user_limiters = {}
    
    def is_allowed(self, user_id):
        # Check global limit first
        if not self.global_limiter.consume():
            return False
        
        # Check user-specific limit
        if user_id not in self.user_limiters:
            self.user_limiters[user_id] = TokenBucket(capacity=100, refill_rate=1)
        
        return self.user_limiters[user_id].consume()
```

### Adaptive Rate Limiting
```python
class AdaptiveRateLimiter:
    def __init__(self):
        self.base_limit = 100
        self.current_limit = self.base_limit
        self.error_rate_threshold = 0.05
        self.success_count = 0
        self.error_count = 0
        self.window_size = 100
    
    def is_allowed(self, key):
        # Adjust limit based on system health
        self._adjust_limit()
        
        limiter = TokenBucket(capacity=self.current_limit, refill_rate=10)
        return limiter.consume()
    
    def record_success(self):
        self.success_count += 1
        self._cleanup_counters()
    
    def record_error(self):
        self.error_count += 1
        self._cleanup_counters()
    
    def _adjust_limit(self):
        total_requests = self.success_count + self.error_count
        if total_requests > 0:
            error_rate = self.error_count / total_requests
            
            if error_rate > self.error_rate_threshold:
                # Reduce limit if error rate is high
                self.current_limit = max(10, self.current_limit * 0.9)
            else:
                # Increase limit if system is healthy
                self.current_limit = min(self.base_limit * 2, self.current_limit * 1.1)
    
    def _cleanup_counters(self):
        total = self.success_count + self.error_count
        if total > self.window_size:
            # Reset counters periodically
            self.success_count = 0
            self.error_count = 0
```

## Implementation in Web Frameworks

### Flask Middleware
```python
from flask import Flask, request, jsonify
from functools import wraps

app = Flask(__name__)
rate_limiter = RedisRateLimiter(redis_client, limit=100, window_size=3600)

def rate_limit(limit=100, window=3600):
    def decorator(f):
        @wraps(f)
        def decorated_function(*args, **kwargs):
            # Use IP address as key
            key = f"rate_limit:{request.remote_addr}"
            
            if not rate_limiter.is_allowed(key):
                return jsonify({
                    'error': 'Rate limit exceeded',
                    'retry_after': window
                }), 429
            
            return f(*args, **kwargs)
        return decorated_function
    return decorator

@app.route('/api/data')
@rate_limit(limit=50, window=3600)
def get_data():
    return jsonify({'data': 'some data'})
```

### Express.js Middleware
```javascript
const redis = require('redis');
const client = redis.createClient();

class RateLimiter {
    constructor(limit, windowMs) {
        this.limit = limit;
        this.windowMs = windowMs;
    }
    
    async isAllowed(key) {
        const now = Date.now();
        const window = Math.floor(now / this.windowMs);
        const redisKey = `${key}:${window}`;
        
        const count = await client.incr(redisKey);
        if (count === 1) {
            await client.expire(redisKey, Math.ceil(this.windowMs / 1000));
        }
        
        return count <= this.limit;
    }
}

const rateLimiter = new RateLimiter(100, 60000); // 100 requests per minute

const rateLimit = async (req, res, next) => {
    const key = `rate_limit:${req.ip}`;
    
    if (await rateLimiter.isAllowed(key)) {
        next();
    } else {
        res.status(429).json({
            error: 'Rate limit exceeded',
            retryAfter: 60
        });
    }
};

app.use('/api/', rateLimit);
```

## Monitoring and Observability

### Metrics Collection
```python
class RateLimiterMetrics:
    def __init__(self, metrics_client):
        self.metrics = metrics_client
    
    def record_request(self, key, allowed):
        self.metrics.counter('rate_limiter.requests.total').labels(
            key=key,
            result='allowed' if allowed else 'rejected'
        ).inc()
    
    def record_bucket_state(self, key, tokens_remaining, capacity):
        self.metrics.gauge('rate_limiter.tokens.remaining').labels(
            key=key
        ).set(tokens_remaining)
        
        utilization = (capacity - tokens_remaining) / capacity
        self.metrics.gauge('rate_limiter.utilization').labels(
            key=key
        ).set(utilization)
```

### Alerting Rules
```yaml
# Prometheus alerting rules
groups:
  - name: rate_limiting
    rules:
      - alert: HighRateLimitRejectionRate
        expr: rate(rate_limiter_requests_total{result="rejected"}[5m]) > 10
        for: 2m
        labels:
          severity: warning
        annotations:
          summary: "High rate limit rejection rate"
          description: "Rate limiter is rejecting more than 10 requests per second"
      
      - alert: RateLimiterHighUtilization
        expr: rate_limiter_utilization > 0.9
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Rate limiter high utilization"
          description: "Rate limiter utilization is above 90%"
```

## Best Practices

### 1. Choose Appropriate Algorithm
- **Token Bucket**: Allow bursts, smooth rate limiting
- **Leaky Bucket**: Smooth output, queue-based
- **Fixed Window**: Simple, but has burst issues
- **Sliding Window**: More accurate, higher memory usage

### 2. Set Reasonable Limits
```python
# Different limits for different user types
RATE_LIMITS = {
    'anonymous': {'requests': 100, 'window': 3600},
    'authenticated': {'requests': 1000, 'window': 3600},
    'premium': {'requests': 10000, 'window': 3600}
}
```

### 3. Provide Clear Error Messages
```python
def rate_limit_response(retry_after):
    return {
        'error': 'Rate limit exceeded',
        'message': 'Too many requests. Please try again later.',
        'retry_after': retry_after,
        'documentation': 'https://api.example.com/docs/rate-limits'
    }, 429
```

### 4. Implement Graceful Degradation
```python
def handle_rate_limited_request(user_id):
    if not rate_limiter.is_allowed(user_id):
        # Return cached data instead of rejecting
        return get_cached_response(user_id)
    
    return process_request(user_id)
```

## Interview Questions

1. **Algorithm Choice**: When would you use token bucket vs leaky bucket?
2. **Distributed Systems**: How do you implement rate limiting across multiple servers?
3. **Storage**: How do you handle rate limiting state in a distributed cache?
4. **Fairness**: How do you ensure fair rate limiting across different user types?
5. **Performance**: What are the performance implications of different algorithms?
6. **Monitoring**: What metrics would you track for rate limiting?

## Real-World Examples

### GitHub API
- **Rate Limits**: 5000 requests/hour for authenticated users
- **Headers**: X-RateLimit-Limit, X-RateLimit-Remaining
- **Algorithm**: Token bucket with different limits per endpoint

### Twitter API
- **Rate Limits**: Different limits per endpoint and user type
- **Windows**: 15-minute windows for most endpoints
- **Algorithm**: Fixed window counter

### AWS API Gateway
- **Throttling**: Request rate and burst limits
- **Per-Key**: Different limits per API key
- **Algorithm**: Token bucket algorithm

Rate limiting is crucial for protecting systems from abuse while ensuring fair access to resources. The choice of algorithm and implementation depends on specific requirements for accuracy, memory usage, and burst handling.