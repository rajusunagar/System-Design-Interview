# Design Social Media Feed (like Facebook/Twitter)

## Problem Statement
Design a social media feed system that shows users a personalized timeline of posts from people they follow.

## Requirements

### Functional Requirements
1. **Post Creation**: Users can create posts (text, images, videos)
2. **Follow System**: Users can follow/unfollow other users
3. **News Feed**: Show personalized feed of posts from followed users
4. **Timeline**: Show user's own posts chronologically
5. **Interactions**: Like, comment, share posts
6. **Real-time Updates**: New posts appear in feed without refresh

### Non-Functional Requirements
1. **Scale**: 1 billion users, 100 million daily active users
2. **Read Heavy**: 100:1 read to write ratio
3. **Latency**: < 200ms for feed generation
4. **Availability**: 99.9% uptime
5. **Consistency**: Eventual consistency acceptable for feeds

## Capacity Estimation

### Users & Posts
- **Total Users**: 1 billion
- **Daily Active Users**: 100 million
- **Posts per user per day**: 2 posts
- **Total posts per day**: 200 million
- **Posts per second**: 200M / (24 × 3600) = 2,315 posts/second

### Feed Reads
- **Feed requests per user**: 10 times/day
- **Total feed requests**: 100M × 10 = 1 billion/day
- **Feed requests per second**: 1B / (24 × 3600) = 11,574 requests/second

### Storage
- **Post size**: 1KB average (text + metadata)
- **Daily storage**: 200M × 1KB = 200GB/day
- **5 years**: 200GB × 365 × 5 = 365TB
- **Media storage**: 20% posts have media (2MB average)
- **Media per day**: 40M × 2MB = 80TB/day

### Following Relationships
- **Average following**: 200 users
- **Total relationships**: 100M × 200 = 20 billion
- **Storage**: 20B × 16 bytes = 320GB

## API Design

### Post Management
```http
POST /api/v1/posts
{
  "content": "Hello World!",
  "mediaUrls": ["image1.jpg", "video1.mp4"],
  "privacy": "public" // public, friends, private
}

Response:
{
  "postId": "123456789",
  "userId": "user123",
  "createdAt": "2024-01-01T12:00:00Z"
}
```

### Feed Generation
```http
GET /api/v1/feed?limit=20&cursor=abc123
Response:
{
  "posts": [
    {
      "postId": "123",
      "userId": "user456",
      "username": "john_doe",
      "content": "Great day!",
      "createdAt": "2024-01-01T11:00:00Z",
      "likes": 150,
      "comments": 25,
      "isLiked": false
    }
  ],
  "nextCursor": "def456",
  "hasMore": true
}
```

### Follow System
```http
POST /api/v1/users/{userId}/follow
DELETE /api/v1/users/{userId}/follow

GET /api/v1/users/{userId}/followers?limit=50
GET /api/v1/users/{userId}/following?limit=50
```

## Database Design

### Users Table
```sql
CREATE TABLE users (
    user_id BIGINT PRIMARY KEY,
    username VARCHAR(50) UNIQUE,
    email VARCHAR(100) UNIQUE,
    display_name VARCHAR(100),
    bio TEXT,
    profile_image_url TEXT,
    follower_count INT DEFAULT 0,
    following_count INT DEFAULT 0,
    created_at TIMESTAMP,
    INDEX idx_username (username)
);
```

### Posts Table
```sql
CREATE TABLE posts (
    post_id BIGINT PRIMARY KEY,
    user_id BIGINT,
    content TEXT,
    media_urls JSON,
    privacy ENUM('public', 'friends', 'private'),
    like_count INT DEFAULT 0,
    comment_count INT DEFAULT 0,
    share_count INT DEFAULT 0,
    created_at TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(user_id),
    INDEX idx_user_time (user_id, created_at),
    INDEX idx_created_at (created_at)
);
```

### Follows Table
```sql
CREATE TABLE follows (
    follower_id BIGINT,
    followee_id BIGINT,
    created_at TIMESTAMP,
    PRIMARY KEY (follower_id, followee_id),
    FOREIGN KEY (follower_id) REFERENCES users(user_id),
    FOREIGN KEY (followee_id) REFERENCES users(user_id),
    INDEX idx_followee (followee_id)
);
```

### Likes Table
```sql
CREATE TABLE likes (
    user_id BIGINT,
    post_id BIGINT,
    created_at TIMESTAMP,
    PRIMARY KEY (user_id, post_id),
    FOREIGN KEY (user_id) REFERENCES users(user_id),
    FOREIGN KEY (post_id) REFERENCES posts(post_id),
    INDEX idx_post_time (post_id, created_at)
);
```

## Feed Generation Approaches

### 1. Pull Model (Fan-out on Read)

#### Algorithm
```python
def generate_feed(user_id, limit=20):
    # Get users that this user follows
    following = get_following_users(user_id)
    
    # Get recent posts from followed users
    posts = []
    for followed_user in following:
        user_posts = get_recent_posts(followed_user, limit=10)
        posts.extend(user_posts)
    
    # Sort by timestamp and return top posts
    posts.sort(key=lambda x: x.created_at, reverse=True)
    return posts[:limit]
```

#### Pros & Cons
- **Pros**: 
  - Simple implementation
  - No storage overhead for feeds
  - Consistent data (always fresh)
- **Cons**: 
  - High latency for users with many followings
  - Expensive for celebrity users
  - Database load on feed requests

### 2. Push Model (Fan-out on Write)

#### Algorithm
```python
def create_post(user_id, content):
    # Create the post
    post = create_post_in_db(user_id, content)
    
    # Get all followers
    followers = get_followers(user_id)
    
    # Add post to each follower's feed
    for follower_id in followers:
        add_to_feed_cache(follower_id, post)
    
    return post

def get_feed(user_id, limit=20):
    # Simply read from pre-computed feed
    return get_feed_from_cache(user_id, limit)
```

#### Feed Storage
```python
# Redis sorted set for each user's feed
# Score = timestamp, Value = post_id
redis_key = f"feed:{user_id}"
redis.zadd(redis_key, {post_id: timestamp})

# Get feed with pagination
def get_feed_from_cache(user_id, limit=20, offset=0):
    redis_key = f"feed:{user_id}"
    post_ids = redis.zrevrange(redis_key, offset, offset + limit - 1)
    return get_posts_by_ids(post_ids)
```

#### Pros & Cons
- **Pros**: 
  - Fast feed generation (pre-computed)
  - Low latency for read operations
  - Scales well for read-heavy workloads
- **Cons**: 
  - High storage overhead
  - Expensive for celebrity users (millions of followers)
  - Potential inconsistency during updates

### 3. Hybrid Model

#### Strategy
```python
def hybrid_feed_generation(user_id):
    # Categorize followed users
    following = get_following_users(user_id)
    celebrities = [u for u in following if u.follower_count > 1000000]
    regular_users = [u for u in following if u.follower_count <= 1000000]
    
    # Pull from celebrities (fan-out on read)
    celebrity_posts = []
    for celebrity in celebrities:
        posts = get_recent_posts(celebrity.user_id, limit=5)
        celebrity_posts.extend(posts)
    
    # Push for regular users (pre-computed feed)
    regular_posts = get_feed_from_cache(user_id, limit=15)
    
    # Merge and sort
    all_posts = celebrity_posts + regular_posts
    all_posts.sort(key=lambda x: x.created_at, reverse=True)
    
    return all_posts[:20]
```

## High-Level Architecture

```
[Mobile/Web Client] → [CDN] → [Load Balancer] → [API Gateway]
                                                      ↓
[Feed Service] ← [Cache Layer] ← [Post Service] ← [User Service]
      ↓              ↓               ↓              ↓
[Feed DB]      [Redis Cache]   [Posts DB]    [Users DB]
      ↓              ↓               ↓              ↓
[Message Queue] ← [Fan-out Service] → [Media Storage]
```

## Detailed Design

### Feed Generation Service
```python
class FeedService:
    def __init__(self):
        self.cache = Redis()
        self.db = Database()
        self.fanout_service = FanoutService()
    
    async def generate_feed(self, user_id, limit=20):
        # Try cache first
        cached_feed = await self.cache.get_feed(user_id, limit)
        if cached_feed:
            return cached_feed
        
        # Generate feed based on strategy
        if self.is_heavy_user(user_id):
            feed = await self.pull_model_feed(user_id, limit)
        else:
            feed = await self.push_model_feed(user_id, limit)
        
        # Cache the result
        await self.cache.set_feed(user_id, feed, ttl=300)
        return feed
    
    def is_heavy_user(self, user_id):
        following_count = self.db.get_following_count(user_id)
        return following_count > 5000
```

### Fan-out Service
```python
class FanoutService:
    def __init__(self):
        self.queue = MessageQueue()
        self.cache = Redis()
    
    async def fanout_post(self, post):
        if post.user.follower_count > 1000000:
            # Celebrity: Don't fan-out, use pull model
            return
        
        # Get followers in batches
        batch_size = 1000
        offset = 0
        
        while True:
            followers = await self.get_followers_batch(
                post.user_id, offset, batch_size
            )
            if not followers:
                break
            
            # Queue fanout job for this batch
            await self.queue.publish('fanout', {
                'post_id': post.post_id,
                'followers': [f.user_id for f in followers]
            })
            
            offset += batch_size
    
    async def process_fanout_batch(self, job):
        post_id = job['post_id']
        followers = job['followers']
        
        # Add post to each follower's feed
        for follower_id in followers:
            await self.cache.add_to_feed(follower_id, post_id)
```

### Caching Strategy

#### Multi-level Caching
```python
class FeedCache:
    def __init__(self):
        self.l1_cache = {}  # Application cache
        self.l2_cache = Redis()  # Distributed cache
    
    async def get_feed(self, user_id, limit):
        # L1 Cache
        cache_key = f"feed:{user_id}"
        if cache_key in self.l1_cache:
            return self.l1_cache[cache_key][:limit]
        
        # L2 Cache
        feed_data = await self.l2_cache.zrevrange(
            f"feed:{user_id}", 0, limit-1
        )
        if feed_data:
            posts = await self.get_posts_by_ids(feed_data)
            self.l1_cache[cache_key] = posts
            return posts
        
        return None
```

#### Cache Warming
```python
class CacheWarmer:
    async def warm_active_users(self):
        # Get list of active users
        active_users = await self.get_active_users(hours=24)
        
        for user_id in active_users:
            if not await self.cache.exists(f"feed:{user_id}"):
                # Generate and cache feed
                feed = await self.feed_service.generate_feed(user_id)
                await self.cache.set_feed(user_id, feed)
```

## Ranking Algorithm

### Simple Chronological
```python
def chronological_ranking(posts):
    return sorted(posts, key=lambda x: x.created_at, reverse=True)
```

### Engagement-based Ranking
```python
def engagement_ranking(posts, user_id):
    def calculate_score(post):
        # Time decay factor
        hours_old = (now() - post.created_at).total_seconds() / 3600
        time_decay = 1 / (1 + hours_old / 24)  # Decay over 24 hours
        
        # Engagement score
        engagement = (
            post.like_count * 1.0 +
            post.comment_count * 2.0 +
            post.share_count * 3.0
        )
        
        # User affinity (how much user interacts with author)
        affinity = get_user_affinity(user_id, post.user_id)
        
        return engagement * time_decay * affinity
    
    return sorted(posts, key=calculate_score, reverse=True)
```

### Machine Learning Ranking
```python
class MLRankingService:
    def __init__(self):
        self.model = load_trained_model()
    
    def rank_posts(self, posts, user_id):
        features = []
        for post in posts:
            feature_vector = self.extract_features(post, user_id)
            features.append(feature_vector)
        
        # Get prediction scores
        scores = self.model.predict(features)
        
        # Sort by predicted engagement probability
        ranked_posts = sorted(
            zip(posts, scores),
            key=lambda x: x[1],
            reverse=True
        )
        
        return [post for post, score in ranked_posts]
    
    def extract_features(self, post, user_id):
        return [
            post.like_count,
            post.comment_count,
            post.share_count,
            (now() - post.created_at).total_seconds(),
            get_user_affinity(user_id, post.user_id),
            len(post.content),
            1 if post.has_media else 0
        ]
```

## Real-time Updates

### WebSocket Implementation
```python
class FeedWebSocket:
    def __init__(self):
        self.connections = {}  # user_id -> websocket
        self.redis_pubsub = Redis().pubsub()
    
    async def connect_user(self, user_id, websocket):
        self.connections[user_id] = websocket
        # Subscribe to user's feed updates
        await self.redis_pubsub.subscribe(f"feed_updates:{user_id}")
    
    async def broadcast_new_post(self, post):
        # Get author's followers
        followers = await self.get_followers(post.user_id)
        
        # Publish to each follower's feed channel
        for follower_id in followers:
            await self.redis.publish(
                f"feed_updates:{follower_id}",
                json.dumps(post.to_dict())
            )
    
    async def handle_feed_update(self, message):
        user_id = message['channel'].split(':')[1]
        if user_id in self.connections:
            websocket = self.connections[user_id]
            await websocket.send(message['data'])
```

## Scaling Strategies

### Database Sharding
```python
def get_user_shard(user_id):
    return user_id % NUM_USER_SHARDS

def get_post_shard(post_id):
    return post_id % NUM_POST_SHARDS

def get_feed_shard(user_id):
    return user_id % NUM_FEED_SHARDS
```

### Read Replicas
```python
class DatabaseRouter:
    def __init__(self):
        self.master = MasterDB()
        self.replicas = [ReplicaDB1(), ReplicaDB2(), ReplicaDB3()]
    
    def write_query(self, query):
        return self.master.execute(query)
    
    def read_query(self, query):
        replica = random.choice(self.replicas)
        return replica.execute(query)
```

### CDN for Media
```python
class MediaService:
    def __init__(self):
        self.s3 = S3Client()
        self.cloudfront = CloudFrontClient()
    
    async def upload_media(self, file, user_id):
        # Upload to S3
        key = f"media/{user_id}/{uuid4()}/{file.name}"
        await self.s3.upload(key, file)
        
        # Return CDN URL
        return f"https://cdn.example.com/{key}"
```

## Monitoring & Analytics

### Key Metrics
```python
class FeedMetrics:
    def __init__(self):
        self.prometheus = PrometheusClient()
    
    def track_feed_generation_time(self, user_id, duration_ms):
        self.prometheus.histogram('feed_generation_duration_ms').observe(duration_ms)
    
    def track_cache_hit_rate(self, cache_type, hit):
        self.prometheus.counter('cache_requests_total').labels(
            cache_type=cache_type,
            result='hit' if hit else 'miss'
        ).inc()
    
    def track_post_engagement(self, post_id, action):
        self.prometheus.counter('post_engagement_total').labels(
            action=action  # like, comment, share
        ).inc()
```

## Interview Discussion Points

1. **Feed Generation**: Pull vs Push vs Hybrid models
2. **Ranking Algorithm**: Chronological vs ML-based ranking
3. **Caching Strategy**: Multi-level caching and cache invalidation
4. **Real-time Updates**: WebSocket vs Server-Sent Events
5. **Database Design**: Sharding strategy and consistency
6. **Celebrity Problem**: Handling users with millions of followers
7. **Content Delivery**: CDN strategy for media files

## Follow-up Questions

1. How would you handle trending topics?
2. How to implement content moderation?
3. How to handle different content types (text, images, videos)?
4. How to implement privacy controls (friends-only posts)?
5. How to prevent spam and fake engagement?
6. How to implement A/B testing for ranking algorithms?

## Real-World Examples

### Facebook News Feed
- **EdgeRank Algorithm**: Affinity, Weight, Time Decay
- **Machine Learning**: Deep learning for ranking
- **Infrastructure**: TAO for social graph, Memcached for caching

### Twitter Timeline
- **Home Timeline**: Algorithmic ranking
- **User Timeline**: Chronological posts
- **Infrastructure**: Manhattan for storage, Redis for caching

### Instagram Feed
- **Algorithm**: Engagement prediction, relationship strength
- **Stories**: Separate feed for ephemeral content
- **Infrastructure**: Cassandra for posts, Redis for feeds