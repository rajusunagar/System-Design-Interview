# Common System Design Interview Questions

## MAANG/FAANG Frequently Asked Questions

### 1. Design a URL Shortener (like bit.ly)
**Companies**: Google, Amazon, Facebook, Microsoft
**Key Concepts**: Hashing, Database design, Caching, Analytics
**Follow-ups**: 
- How to handle custom URLs?
- How to implement analytics?
- How to handle 10x more traffic?

### 2. Design a Chat System (like WhatsApp/Slack)
**Companies**: Facebook, Microsoft, Uber, Airbnb
**Key Concepts**: WebSockets, Message queues, Real-time systems
**Follow-ups**:
- How to handle group chats?
- How to implement message history?
- How to handle offline users?

### 3. Design a Social Media Feed (like Facebook/Twitter)
**Companies**: Facebook, Twitter, Instagram, LinkedIn
**Key Concepts**: Fan-out strategies, Caching, Ranking algorithms
**Follow-ups**:
- How to handle celebrity users?
- How to implement trending topics?
- How to personalize the feed?

### 4. Design a Video Streaming Service (like YouTube/Netflix)
**Companies**: Netflix, YouTube, Amazon, Apple
**Key Concepts**: CDN, Video encoding, Metadata storage
**Follow-ups**:
- How to handle live streaming?
- How to implement video recommendations?
- How to handle different video qualities?

### 5. Design a Search Engine (like Google)
**Companies**: Google, Microsoft, Amazon
**Key Concepts**: Web crawling, Indexing, Ranking algorithms
**Follow-ups**:
- How to handle real-time updates?
- How to implement autocomplete?
- How to handle different languages?

### 6. Design a Ride Sharing Service (like Uber/Lyft)
**Companies**: Uber, Lyft, Amazon, Google
**Key Concepts**: Geospatial indexing, Real-time matching, Location services
**Follow-ups**:
- How to handle surge pricing?
- How to optimize driver-rider matching?
- How to handle GPS inaccuracies?

### 7. Design an E-commerce System (like Amazon)
**Companies**: Amazon, eBay, Shopify, Walmart
**Key Concepts**: Inventory management, Payment processing, Order fulfillment
**Follow-ups**:
- How to handle flash sales?
- How to implement product recommendations?
- How to handle returns and refunds?

### 8. Design a Payment System (like PayPal/Stripe)
**Companies**: PayPal, Stripe, Square, Amazon
**Key Concepts**: Transaction processing, Security, Compliance
**Follow-ups**:
- How to handle fraud detection?
- How to ensure transaction consistency?
- How to handle international payments?

### 9. Design a Notification System
**Companies**: Facebook, Google, Apple, Microsoft
**Key Concepts**: Push notifications, Message queues, Fan-out
**Follow-ups**:
- How to handle different notification types?
- How to implement notification preferences?
- How to handle notification delivery failures?

### 10. Design a Distributed Cache (like Redis/Memcached)
**Companies**: Redis Labs, Amazon, Google, Facebook
**Key Concepts**: Consistent hashing, Replication, Eviction policies
**Follow-ups**:
- How to handle cache invalidation?
- How to implement cache warming?
- How to handle cache failures?

## Question Categories by Difficulty

### Beginner Level (0-2 years experience)
1. **Design a Key-Value Store**
   - Focus: Basic database concepts, simple APIs
   - Time: 30-40 minutes

2. **Design a Web Crawler**
   - Focus: BFS/DFS, URL processing, basic scaling
   - Time: 35-45 minutes

3. **Design a Rate Limiter**
   - Focus: Algorithms, basic distributed concepts
   - Time: 30-40 minutes

### Intermediate Level (2-5 years experience)
1. **Design Instagram**
   - Focus: Image storage, feed generation, caching
   - Time: 45-60 minutes

2. **Design Dropbox**
   - Focus: File synchronization, conflict resolution
   - Time: 45-60 minutes

3. **Design a Load Balancer**
   - Focus: Algorithms, health checks, high availability
   - Time: 40-50 minutes

### Advanced Level (5+ years experience)
1. **Design Google Maps**
   - Focus: Geospatial data, routing algorithms, real-time updates
   - Time: 60-75 minutes

2. **Design a Distributed Database**
   - Focus: Consistency, partitioning, replication
   - Time: 60-75 minutes

3. **Design a Stock Trading System**
   - Focus: Low latency, consistency, fault tolerance
   - Time: 60-75 minutes

## Company-Specific Patterns

### Google
**Common Questions**:
- Design Google Search
- Design Google Maps
- Design YouTube
- Design Gmail

**Focus Areas**:
- Massive scale (billions of users)
- Global distribution
- Machine learning integration
- Real-time processing

### Amazon
**Common Questions**:
- Design Amazon.com
- Design AWS S3
- Design Kindle
- Design Amazon Prime Video

**Focus Areas**:
- E-commerce workflows
- Cloud services
- Cost optimization
- Reliability and availability

### Facebook (Meta)
**Common Questions**:
- Design Facebook News Feed
- Design Instagram
- Design WhatsApp
- Design Facebook Messenger

**Focus Areas**:
- Social graphs
- Real-time communication
- Content delivery
- User engagement

### Netflix
**Common Questions**:
- Design Netflix streaming
- Design recommendation system
- Design content delivery
- Design user analytics

**Focus Areas**:
- Video streaming
- Content recommendation
- Global CDN
- A/B testing

### Uber
**Common Questions**:
- Design Uber ride matching
- Design Uber Eats
- Design surge pricing
- Design driver tracking

**Focus Areas**:
- Real-time location services
- Geospatial algorithms
- Dynamic pricing
- Mobile-first design

## Question Variations and Follow-ups

### URL Shortener Variations
1. **Basic**: Simple URL shortening
2. **Analytics**: Add click tracking and analytics
3. **Custom URLs**: Allow custom short URLs
4. **Expiration**: Add URL expiration
5. **QR Codes**: Generate QR codes for URLs

### Chat System Variations
1. **1-on-1 Chat**: Simple messaging between two users
2. **Group Chat**: Multiple users in a conversation
3. **Channels**: Public channels like Slack
4. **Voice/Video**: Add calling functionality
5. **File Sharing**: Support media and file uploads

### Social Media Variations
1. **Timeline**: User's own posts
2. **News Feed**: Posts from followed users
3. **Stories**: Ephemeral content
4. **Live Streaming**: Real-time video broadcasting
5. **Trending**: Popular content discovery

## Technical Deep Dives

### Database Questions
- "How would you design the database schema?"
- "SQL or NoSQL? Why?"
- "How would you handle database scaling?"
- "What about data consistency?"

### Caching Questions
- "Where would you add caching?"
- "What caching strategy would you use?"
- "How do you handle cache invalidation?"
- "What about cache warming?"

### Scalability Questions
- "How does this scale to 1 million users?"
- "What about 1 billion users?"
- "Where are the bottlenecks?"
- "How do you handle traffic spikes?"

### Reliability Questions
- "What happens if a server crashes?"
- "How do you ensure data durability?"
- "What's your disaster recovery plan?"
- "How do you handle network partitions?"

## Red Flags to Avoid

### Technical Red Flags
1. **Overengineering**: Adding unnecessary complexity
2. **Underengineering**: Ignoring scalability requirements
3. **Single Point of Failure**: Not considering redundancy
4. **Ignoring Constraints**: Not addressing given requirements
5. **Poor API Design**: Inconsistent or unclear interfaces

### Communication Red Flags
1. **Not Asking Questions**: Assuming requirements
2. **Going Silent**: Not explaining thought process
3. **Ignoring Feedback**: Not incorporating interviewer input
4. **Time Management**: Spending too much time on one area
5. **Not Prioritizing**: Treating all features equally

## Preparation Strategy

### Study Plan (4-6 weeks)
**Week 1-2**: Fundamentals
- System design basics
- Database concepts
- Caching strategies
- Load balancing

**Week 3-4**: Components and Patterns
- Message queues
- Microservices
- Design patterns
- Security concepts

**Week 5-6**: Practice and Mock Interviews
- Solve 2-3 problems daily
- Time yourself
- Practice drawing diagrams
- Mock interviews with peers

### Daily Practice Routine
1. **Morning (30 minutes)**: Review one concept
2. **Afternoon (45 minutes)**: Solve one design problem
3. **Evening (15 minutes)**: Review and reflect

### Resources for Practice
- **Books**: "Designing Data-Intensive Applications"
- **Online**: System Design Primer, High Scalability
- **Videos**: YouTube system design channels
- **Practice**: LeetCode, Pramp, InterviewBit

## Sample Interview Timeline

### 45-Minute Interview
- **0-5 min**: Problem clarification and requirements
- **5-15 min**: High-level architecture design
- **15-25 min**: Detailed component design
- **25-35 min**: Database and API design
- **35-40 min**: Scaling and optimization
- **40-45 min**: Questions and wrap-up

### 60-Minute Interview
- **0-8 min**: Problem clarification and requirements
- **8-20 min**: High-level architecture design
- **20-35 min**: Detailed component design
- **35-45 min**: Database and API design
- **45-55 min**: Scaling, monitoring, and edge cases
- **55-60 min**: Questions and wrap-up

## Success Metrics

### What Interviewers Look For
1. **Problem Solving**: Systematic approach to complex problems
2. **Technical Knowledge**: Understanding of system components
3. **Communication**: Clear explanation of design decisions
4. **Trade-off Analysis**: Understanding pros and cons
5. **Scalability Thinking**: Designing for growth

### Scoring Criteria
- **Requirements Gathering**: Did you ask the right questions?
- **Architecture Design**: Is the high-level design sound?
- **Component Design**: Are individual components well-designed?
- **Scalability**: Can the system handle the required scale?
- **Communication**: Were you clear and organized?

Remember: System design interviews are about demonstrating your thought process and engineering judgment, not finding the "perfect" solution. Focus on clear communication, reasonable trade-offs, and systematic problem-solving.