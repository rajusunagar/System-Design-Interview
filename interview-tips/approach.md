# System Design Interview Approach & Framework

## The RADIO Framework

### R - Requirements Clarification (5-10 minutes)

#### Functional Requirements
- **Core Features**: What should the system do?
- **User Actions**: What can users do with the system?
- **Business Logic**: Any specific business rules?

**Example Questions:**
- "Should users be able to edit their posts?"
- "Do we need real-time notifications?"
- "Should the system support mobile apps?"

#### Non-Functional Requirements
- **Scale**: How many users? How much data?
- **Performance**: Latency and throughput requirements
- **Availability**: Uptime requirements (99.9%, 99.99%?)
- **Consistency**: Strong vs eventual consistency needs

**Example Questions:**
- "How many daily active users do we expect?"
- "What's the acceptable response time?"
- "Is it okay to show slightly stale data?"

### A - Architecture & High-Level Design (10-15 minutes)

#### Start Simple
```
[Client] → [Load Balancer] → [Web Server] → [Database]
```

#### Add Components Gradually
- **Load Balancer**: For high availability
- **Cache**: For performance
- **CDN**: For global reach
- **Message Queue**: For async processing

#### Draw the Architecture
- Use boxes and arrows
- Label each component
- Show data flow direction
- Keep it clean and readable

### D - Data Model Design (10-15 minutes)

#### Database Choice
- **SQL**: ACID properties, complex queries, structured data
- **NoSQL**: Scalability, flexibility, unstructured data

#### Schema Design
```sql
-- Example: Social Media Post
CREATE TABLE posts (
    post_id BIGINT PRIMARY KEY,
    user_id BIGINT,
    content TEXT,
    created_at TIMESTAMP,
    INDEX idx_user_time (user_id, created_at)
);
```

#### Key Considerations
- **Relationships**: Foreign keys and joins
- **Indexing**: Query optimization
- **Partitioning**: Horizontal scaling strategy

### I - Interface Design (5-10 minutes)

#### API Endpoints
```http
POST /api/v1/posts
GET /api/v1/posts/{postId}
GET /api/v1/users/{userId}/posts
DELETE /api/v1/posts/{postId}
```

#### Request/Response Format
```json
POST /api/v1/posts
{
  "content": "Hello World!",
  "mediaUrls": ["image1.jpg"]
}

Response:
{
  "postId": "123456",
  "createdAt": "2024-01-01T12:00:00Z"
}
```

### O - Optimize & Scale (15-20 minutes)

#### Identify Bottlenecks
- **Database**: Usually the first bottleneck
- **Network**: Bandwidth limitations
- **CPU**: Processing power
- **Memory**: Cache size limitations

#### Scaling Solutions
- **Horizontal Scaling**: Add more servers
- **Caching**: Multiple cache layers
- **Database Scaling**: Sharding, read replicas
- **CDN**: Global content distribution

## Step-by-Step Process

### 1. Listen Carefully (2 minutes)
- Take notes on the problem statement
- Don't jump to solutions immediately
- Ask for clarification if needed

### 2. Clarify Requirements (5-8 minutes)
```
Interviewer: "Design a social media platform"

You: "Let me clarify the requirements:
- Should users be able to post text, images, and videos?
- Do we need a news feed algorithm?
- Should users be able to follow each other?
- What's the expected scale - millions or billions of users?"
```

### 3. Estimate Scale (3-5 minutes)
```
Assumptions:
- 1 billion users
- 100 million daily active users
- Each user posts once per day on average
- Each post is 1KB on average

Calculations:
- Write QPS: 100M posts/day = 1,157 posts/second
- Storage: 100M × 1KB = 100GB/day
- Read QPS: Assume 100:1 read/write ratio = 115,700/second
```

### 4. High-Level Design (10 minutes)
- Start with simple architecture
- Add components one by one
- Explain the purpose of each component
- Show data flow

### 5. Database Design (10 minutes)
- Choose database type (SQL vs NoSQL)
- Design main tables/collections
- Define relationships
- Consider indexing strategy

### 6. API Design (5 minutes)
- Define main endpoints
- Show request/response format
- Consider authentication
- Think about rate limiting

### 7. Deep Dive (15 minutes)
- Pick 1-2 components to discuss in detail
- Address scalability concerns
- Discuss trade-offs
- Consider edge cases

### 8. Scale & Optimize (10 minutes)
- Identify bottlenecks
- Propose scaling solutions
- Discuss caching strategies
- Consider monitoring and alerting

## Common Mistakes to Avoid

### 1. Jumping to Solutions Too Quickly
❌ **Wrong**: Start designing immediately
✅ **Right**: Clarify requirements first

### 2. Over-Engineering
❌ **Wrong**: Add every possible feature and component
✅ **Right**: Start simple, add complexity gradually

### 3. Ignoring Scale
❌ **Wrong**: Design for small scale only
✅ **Right**: Consider scalability from the beginning

### 4. Not Considering Trade-offs
❌ **Wrong**: Present solutions without discussing alternatives
✅ **Right**: Explain why you chose one approach over another

### 5. Poor Communication
❌ **Wrong**: Design in silence
✅ **Right**: Think out loud, explain your reasoning

## Time Management Tips

### 45-Minute Interview Breakdown
- **Requirements**: 8 minutes
- **High-level design**: 12 minutes
- **Database design**: 8 minutes
- **API design**: 5 minutes
- **Deep dive**: 10 minutes
- **Q&A**: 2 minutes

### Stay on Track
- Set mental timers for each section
- If running over, summarize and move on
- Prioritize core functionality over nice-to-haves
- Leave time for questions at the end

## Communication Best Practices

### Think Out Loud
```
"I'm thinking about the database design now. 
Since we need to support billions of posts, 
I'm considering NoSQL for better horizontal scaling. 
Let me think about the trade-offs..."
```

### Ask for Feedback
```
"Does this architecture make sense so far?"
"Should I dive deeper into the caching layer?"
"Are there any specific areas you'd like me to focus on?"
```

### Acknowledge Constraints
```
"Given the time constraints, I'll focus on the core features.
In a real implementation, we'd also need to consider..."
```

## Drawing Tips

### Use Simple Shapes
- **Rectangles**: Services, databases, caches
- **Circles**: Load balancers, users
- **Arrows**: Data flow, API calls
- **Cylinders**: Databases (if you can draw them)

### Label Everything
- Service names
- Database types
- Protocol names (HTTP, TCP, etc.)
- Data flow direction

### Keep It Organized
- Left to right flow (user → system)
- Group related components
- Use consistent spacing
- Don't overcrowd the diagram

## Follow-up Questions Preparation

### Performance Questions
- "How would you handle 10x more traffic?"
- "What if the database becomes the bottleneck?"
- "How would you reduce latency?"

### Reliability Questions
- "What happens if a server crashes?"
- "How do you ensure data consistency?"
- "How do you handle network partitions?"

### Security Questions
- "How do you authenticate users?"
- "How do you prevent spam and abuse?"
- "How do you handle sensitive data?"

## Practice Recommendations

### Mock Interviews
- Practice with peers or online platforms
- Record yourself explaining designs
- Get feedback on communication style
- Time yourself regularly

### Study Real Systems
- Read engineering blogs (Netflix, Uber, Airbnb)
- Understand how real companies solve problems
- Learn from their trade-offs and decisions
- Follow system design newsletters

### Build Mental Models
- Understand common patterns
- Know when to use each component
- Practice capacity estimation
- Memorize key numbers (latency, throughput)

Remember: System design interviews test your ability to think through complex problems, make reasonable trade-offs, and communicate effectively. Focus on demonstrating your thought process rather than finding the "perfect" solution.