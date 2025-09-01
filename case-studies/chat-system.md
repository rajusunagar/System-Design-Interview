# Design Chat System (like WhatsApp/Slack)

## Problem Statement
Design a real-time chat system that supports one-on-one messaging, group chats, and online presence indicators.

## Requirements

### Functional Requirements
1. **One-on-one chat**: Send/receive messages between two users
2. **Group chat**: Multiple users in a conversation
3. **Online status**: Show user online/offline status
4. **Message history**: Store and retrieve chat history
5. **Push notifications**: Notify users of new messages
6. **Media sharing**: Send images, files, videos
7. **Message status**: Sent, delivered, read receipts

### Non-Functional Requirements
1. **Scale**: 500M daily active users
2. **Latency**: < 100ms message delivery
3. **Availability**: 99.9% uptime
4. **Consistency**: Messages delivered in order
5. **Security**: End-to-end encryption
6. **Storage**: 5 years of message history

## Capacity Estimation

### Users & Messages
- **Daily Active Users**: 500M
- **Messages per user**: 40 messages/day
- **Total messages**: 20B messages/day
- **Peak QPS**: 20B / (24 × 3600) × 3 = 694K messages/second

### Storage
- **Message size**: 100 bytes average
- **Daily storage**: 20B × 100 bytes = 2TB/day
- **5 years**: 2TB × 365 × 5 = 3.6PB
- **Media**: 10% messages have media (10MB average)
- **Media storage**: 2B × 10MB = 20TB/day

### Bandwidth
- **Incoming**: 694K × 100 bytes = 69MB/s
- **Outgoing**: 69MB/s × 2 (delivery to sender & receiver) = 138MB/s
- **Media**: Additional 23GB/s for media content

## API Design

### Send Message
```http
POST /api/v1/messages
{
  "chatId": "chat_123",
  "senderId": "user_456",
  "content": "Hello World!",
  "messageType": "text", // text, image, file, video
  "replyToId": "msg_789" // optional
}

Response:
{
  "messageId": "msg_abc123",
  "timestamp": "2024-01-01T12:00:00Z",
  "status": "sent"
}
```

### Get Messages
```http
GET /api/v1/chats/chat_123/messages?limit=50&before=msg_xyz
Response:
{
  "messages": [...],
  "hasMore": true,
  "nextCursor": "msg_def456"
}
```

### WebSocket Events
```javascript
// Connect
ws://chat.example.com/ws?userId=user_123&token=jwt_token

// Message events
{
  "type": "message",
  "data": {
    "messageId": "msg_123",
    "chatId": "chat_456",
    "senderId": "user_789",
    "content": "Hello!",
    "timestamp": "2024-01-01T12:00:00Z"
  }
}

// Presence events
{
  "type": "presence",
  "data": {
    "userId": "user_123",
    "status": "online", // online, offline, away
    "lastSeen": "2024-01-01T12:00:00Z"
  }
}
```

## Database Design

### Users Table
```sql
CREATE TABLE users (
    user_id BIGINT PRIMARY KEY,
    username VARCHAR(50) UNIQUE,
    email VARCHAR(100) UNIQUE,
    phone VARCHAR(20),
    profile_image_url TEXT,
    created_at TIMESTAMP,
    last_seen TIMESTAMP,
    status ENUM('online', 'offline', 'away')
);
```

### Chats Table
```sql
CREATE TABLE chats (
    chat_id BIGINT PRIMARY KEY,
    chat_type ENUM('direct', 'group'),
    name VARCHAR(100), -- for group chats
    created_by BIGINT,
    created_at TIMESTAMP,
    updated_at TIMESTAMP,
    FOREIGN KEY (created_by) REFERENCES users(user_id)
);
```

### Messages Table
```sql
CREATE TABLE messages (
    message_id BIGINT PRIMARY KEY,
    chat_id BIGINT,
    sender_id BIGINT,
    content TEXT,
    message_type ENUM('text', 'image', 'file', 'video'),
    reply_to_id BIGINT,
    created_at TIMESTAMP,
    FOREIGN KEY (chat_id) REFERENCES chats(chat_id),
    FOREIGN KEY (sender_id) REFERENCES users(user_id),
    INDEX idx_chat_time (chat_id, created_at)
);
```

### Chat Participants
```sql
CREATE TABLE chat_participants (
    chat_id BIGINT,
    user_id BIGINT,
    joined_at TIMESTAMP,
    role ENUM('admin', 'member'),
    last_read_message_id BIGINT,
    PRIMARY KEY (chat_id, user_id),
    FOREIGN KEY (chat_id) REFERENCES chats(chat_id),
    FOREIGN KEY (user_id) REFERENCES users(user_id)
);
```

## High-Level Architecture

```
[Mobile/Web Client] ↔ [Load Balancer] ↔ [API Gateway]
                                            ↓
[WebSocket Servers] ↔ [Message Queue] ↔ [Chat Service]
        ↓                    ↓              ↓
[Presence Service] ↔ [Notification] ↔ [User Service]
        ↓                    ↓              ↓
    [Redis]           [Push Gateway]   [User DB]
                           ↓
                    [Message DB] ↔ [Media Storage]
```

## Detailed Design

### Real-time Communication

#### WebSocket Connection Management
```python
class ConnectionManager:
    def __init__(self):
        self.connections = {}  # user_id -> websocket
        
    async def connect(self, user_id, websocket):
        self.connections[user_id] = websocket
        await self.update_presence(user_id, "online")
        
    async def disconnect(self, user_id):
        if user_id in self.connections:
            del self.connections[user_id]
            await self.update_presence(user_id, "offline")
```

#### Message Delivery
```python
async def send_message(message):
    # Store message in database
    message_id = await store_message(message)
    
    # Get chat participants
    participants = await get_chat_participants(message.chat_id)
    
    # Send to online users via WebSocket
    for user_id in participants:
        if user_id in connection_manager.connections:
            await connection_manager.send_to_user(user_id, message)
        else:
            # Queue for offline delivery
            await queue_offline_message(user_id, message)
```

### Message Ordering

#### Sequence Numbers
```python
class MessageSequencer:
    def __init__(self):
        self.sequences = {}  # chat_id -> sequence_number
        
    def get_next_sequence(self, chat_id):
        if chat_id not in self.sequences:
            self.sequences[chat_id] = 0
        self.sequences[chat_id] += 1
        return self.sequences[chat_id]
```

#### Vector Clocks (for distributed systems)
```python
class VectorClock:
    def __init__(self, node_id):
        self.node_id = node_id
        self.clock = {node_id: 0}
        
    def tick(self):
        self.clock[self.node_id] += 1
        
    def update(self, other_clock):
        for node, time in other_clock.items():
            self.clock[node] = max(self.clock.get(node, 0), time)
        self.tick()
```

### Presence System

#### Online Status Tracking
```python
class PresenceService:
    def __init__(self):
        self.redis = Redis()
        
    async def set_online(self, user_id):
        await self.redis.setex(f"presence:{user_id}", 300, "online")
        await self.broadcast_presence(user_id, "online")
        
    async def heartbeat(self, user_id):
        await self.redis.expire(f"presence:{user_id}", 300)
        
    async def check_offline_users(self):
        # Periodic job to mark users offline
        expired_keys = await self.redis.scan_iter("presence:*")
        for key in expired_keys:
            user_id = key.split(":")[1]
            await self.broadcast_presence(user_id, "offline")
```

### Message Storage Strategy

#### Database Sharding
```python
def get_message_shard(chat_id):
    return chat_id % NUM_MESSAGE_SHARDS

def get_user_shard(user_id):
    return user_id % NUM_USER_SHARDS
```

#### Message Archival
```python
class MessageArchival:
    def archive_old_messages(self, days=365):
        cutoff_date = datetime.now() - timedelta(days=days)
        
        # Move old messages to cold storage
        old_messages = get_messages_before(cutoff_date)
        store_in_cold_storage(old_messages)
        delete_from_hot_storage(old_messages)
```

## Scaling Strategies

### Horizontal Scaling

#### WebSocket Server Scaling
- **Consistent Hashing**: Route users to specific servers
- **Session Affinity**: Sticky sessions for WebSocket connections
- **Server Discovery**: Service registry for dynamic scaling

#### Database Scaling
- **Read Replicas**: Scale read operations
- **Sharding**: Partition by chat_id or user_id
- **Caching**: Redis for recent messages and presence

### Message Queue Architecture
```
[Chat Service] → [Kafka] → [Message Processor] → [Database]
                    ↓
              [Notification Service]
                    ↓
              [Push Notifications]
```

### Caching Strategy

#### Multi-level Caching
```python
class MessageCache:
    def __init__(self):
        self.l1_cache = {}  # In-memory cache
        self.l2_cache = Redis()  # Distributed cache
        
    async def get_messages(self, chat_id, limit=50):
        # Try L1 cache first
        if chat_id in self.l1_cache:
            return self.l1_cache[chat_id][:limit]
            
        # Try L2 cache
        cached = await self.l2_cache.get(f"messages:{chat_id}")
        if cached:
            messages = json.loads(cached)
            self.l1_cache[chat_id] = messages
            return messages[:limit]
            
        # Fetch from database
        messages = await db.get_messages(chat_id, limit)
        await self.cache_messages(chat_id, messages)
        return messages
```

## Security & Privacy

### End-to-End Encryption
```python
class E2EEncryption:
    def generate_key_pair(self):
        private_key = rsa.generate_private_key(
            public_exponent=65537,
            key_size=2048
        )
        public_key = private_key.public_key()
        return private_key, public_key
        
    def encrypt_message(self, message, recipient_public_key):
        encrypted = recipient_public_key.encrypt(
            message.encode(),
            padding.OAEP(
                mgf=padding.MGF1(algorithm=hashes.SHA256()),
                algorithm=hashes.SHA256(),
                label=None
            )
        )
        return encrypted
```

### Authentication & Authorization
```python
class ChatAuthorization:
    def can_send_message(self, user_id, chat_id):
        return user_id in get_chat_participants(chat_id)
        
    def can_read_messages(self, user_id, chat_id):
        participant = get_participant(chat_id, user_id)
        return participant and participant.joined_at <= message.created_at
```

## Advanced Features

### Message Search
```python
class MessageSearch:
    def __init__(self):
        self.elasticsearch = Elasticsearch()
        
    async def index_message(self, message):
        doc = {
            'content': message.content,
            'chat_id': message.chat_id,
            'sender_id': message.sender_id,
            'timestamp': message.created_at
        }
        await self.elasticsearch.index(
            index='messages',
            id=message.message_id,
            body=doc
        )
```

### File Sharing
```python
class FileService:
    def __init__(self):
        self.s3 = boto3.client('s3')
        
    async def upload_file(self, file, user_id):
        # Generate unique filename
        filename = f"{user_id}/{uuid4()}/{file.filename}"
        
        # Upload to S3
        self.s3.upload_fileobj(file, 'chat-files', filename)
        
        # Generate signed URL
        url = self.s3.generate_presigned_url(
            'get_object',
            Params={'Bucket': 'chat-files', 'Key': filename},
            ExpiresIn=3600
        )
        return url
```

## Monitoring & Analytics

### Key Metrics
- **Message delivery rate**: Messages/second
- **WebSocket connections**: Active connections
- **Message latency**: End-to-end delivery time
- **Presence accuracy**: Online status correctness
- **Storage growth**: Database size growth rate

### Real-time Monitoring
```python
class ChatMetrics:
    def __init__(self):
        self.prometheus = PrometheusMetrics()
        
    def record_message_sent(self, chat_type):
        self.prometheus.counter('messages_sent_total').labels(
            chat_type=chat_type
        ).inc()
        
    def record_message_latency(self, latency_ms):
        self.prometheus.histogram('message_latency_ms').observe(latency_ms)
```

## Interview Discussion Points

1. **Real-time Communication**: WebSocket vs Server-Sent Events vs Long Polling
2. **Message Ordering**: How to ensure message order in distributed systems
3. **Presence System**: Challenges in tracking online status at scale
4. **Database Design**: SQL vs NoSQL for chat systems
5. **Caching Strategy**: What to cache and cache invalidation
6. **Security**: End-to-end encryption implementation
7. **Scaling**: Handling millions of concurrent connections

## Follow-up Questions

1. How would you implement message reactions (like/emoji)?
2. How to handle message editing and deletion?
3. How to implement voice/video calling?
4. How to handle spam and abuse?
5. How to implement message threading (like Slack)?
6. How to sync messages across multiple devices?

## Real-World Examples

### WhatsApp
- **XMPP Protocol**: Modified for mobile
- **Erlang**: Backend for concurrency
- **FreeBSD**: Operating system choice
- **End-to-end encryption**: Signal protocol

### Slack
- **WebSocket**: Real-time communication
- **Elasticsearch**: Message search
- **MySQL**: Primary database
- **Redis**: Caching and presence

### Discord
- **Elixir**: Backend language
- **Cassandra**: Message storage
- **Redis**: Caching
- **Voice channels**: WebRTC integration