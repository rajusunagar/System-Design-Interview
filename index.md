# System Design Interview Preparation - Complete Guide

## 🎯 Quick Start

### For Beginners (0-2 years)
1. [Scalability](./fundamentals/scalability.md) → [Load Balancers](./components/load-balancers.md) → [URL Shortener](./case-studies/url-shortener.md)
2. [Databases](./components/databases.md) → [SQL vs NoSQL](./data-storage/sql-vs-nosql.md)
3. [Interview Approach](./interview-tips/approach.md) → [Common Questions](./interview-tips/questions.md)

### For Experienced (3+ years)
1. [CAP Theorem](./fundamentals/cap-theorem.md) → [Circuit Breaker](./patterns/circuit-breaker.md)
2. [Chat System](./case-studies/chat-system.md) → [Social Media Feed](./case-studies/social-media-feed.md)
3. [Rate Limiting](./security/rate-limiting.md) → [Message Queues](./components/message-queues.md)

## 📚 Complete Curriculum

### 🏗️ Fundamentals
| Topic | Difficulty | Time | Key Concepts |
|-------|------------|------|--------------|
| [Scalability](./fundamentals/scalability.md) | ⭐⭐ | 30min | Horizontal/Vertical scaling, Bottlenecks |
| [Reliability](./fundamentals/reliability.md) | ⭐⭐⭐ | 45min | MTBF, MTTR, Fault tolerance |
| [CAP Theorem](./fundamentals/cap-theorem.md) | ⭐⭐⭐ | 30min | Consistency, Availability, Partition tolerance |

### 🔧 System Components
| Component | Difficulty | Time | Use Cases |
|-----------|------------|------|-----------|
| [Load Balancers](./components/load-balancers.md) | ⭐⭐ | 40min | Traffic distribution, High availability |
| [Databases](./components/databases.md) | ⭐⭐⭐ | 60min | RDBMS, NoSQL, Sharding, Replication |
| [Caching](./components/caching.md) | ⭐⭐⭐ | 45min | Multi-level caching, Cache strategies |
| [Message Queues](./components/message-queues.md) | ⭐⭐⭐⭐ | 50min | Async processing, Event-driven architecture |

### 📊 Data Storage
| Topic | Difficulty | Time | Key Decisions |
|-------|------------|------|---------------|
| [SQL vs NoSQL](./data-storage/sql-vs-nosql.md) | ⭐⭐⭐ | 45min | Database selection criteria |

### 🚀 Design Patterns
| Pattern | Difficulty | Time | Problem Solved |
|---------|------------|------|----------------|
| [Circuit Breaker](./patterns/circuit-breaker.md) | ⭐⭐⭐⭐ | 40min | Fault tolerance, Cascading failures |

### 🔒 Security
| Topic | Difficulty | Time | Applications |
|-------|------------|------|--------------|
| [Rate Limiting](./security/rate-limiting.md) | ⭐⭐⭐ | 35min | API protection, DDoS prevention |

### 🎯 Case Studies (Interview Questions)
| System | Companies | Difficulty | Time | Key Concepts |
|--------|-----------|------------|------|--------------|
| [URL Shortener](./case-studies/url-shortener.md) | Google, Amazon | ⭐⭐ | 45min | Hashing, Caching, Analytics |
| [Chat System](./case-studies/chat-system.md) | Facebook, Slack | ⭐⭐⭐ | 60min | WebSockets, Real-time, Message queues |
| [Social Media Feed](./case-studies/social-media-feed.md) | Facebook, Twitter | ⭐⭐⭐⭐ | 75min | Fan-out, Ranking, Caching |

### 📝 Interview Success
| Resource | Purpose | Time |
|----------|---------|------|
| [RADIO Framework](./interview-tips/approach.md) | Structured approach | 20min |
| [Common Questions](./interview-tips/questions.md) | Practice problems | 60min |

## 🏢 Company-Specific Prep

### Google/Alphabet
**Focus**: Massive scale, global distribution
- [Scalability](./fundamentals/scalability.md) + [Social Media Feed](./case-studies/social-media-feed.md)
- **Common Q**: Design YouTube, Google Search, Gmail

### Amazon/AWS
**Focus**: E-commerce, cloud services, reliability
- [Reliability](./fundamentals/reliability.md) + [Databases](./components/databases.md)
- **Common Q**: Design Amazon.com, S3, Prime Video

### Meta/Facebook
**Focus**: Social graphs, real-time communication
- [Chat System](./case-studies/chat-system.md) + [Social Media Feed](./case-studies/social-media-feed.md)
- **Common Q**: Design Facebook, Instagram, WhatsApp

### Netflix
**Focus**: Video streaming, recommendations, CDN
- [Caching](./components/caching.md) + [Circuit Breaker](./patterns/circuit-breaker.md)
- **Common Q**: Design Netflix, recommendation engine

### Uber/Lyft
**Focus**: Real-time systems, geospatial data
- [Message Queues](./components/message-queues.md) + [Rate Limiting](./security/rate-limiting.md)
- **Common Q**: Design ride-sharing, real-time tracking

## ⚡ 30-Day Study Plan

### Week 1: Foundations
- **Day 1-2**: [Scalability](./fundamentals/scalability.md) + [Load Balancers](./components/load-balancers.md)
- **Day 3-4**: [Databases](./components/databases.md) + [SQL vs NoSQL](./data-storage/sql-vs-nosql.md)
- **Day 5-6**: [Caching](./components/caching.md) + [Reliability](./fundamentals/reliability.md)
- **Day 7**: Review + [Interview Approach](./interview-tips/approach.md)

### Week 2: Core Components
- **Day 8-9**: [Message Queues](./components/message-queues.md)
- **Day 10-11**: [CAP Theorem](./fundamentals/cap-theorem.md) + [Circuit Breaker](./patterns/circuit-breaker.md)
- **Day 12-13**: [Rate Limiting](./security/rate-limiting.md)
- **Day 14**: Practice problems from [Common Questions](./interview-tips/questions.md)

### Week 3: Case Studies
- **Day 15-17**: [URL Shortener](./case-studies/url-shortener.md) (practice 3x)
- **Day 18-20**: [Chat System](./case-studies/chat-system.md) (practice 3x)
- **Day 21**: Review and identify weak areas

### Week 4: Advanced + Mock Interviews
- **Day 22-24**: [Social Media Feed](./case-studies/social-media-feed.md) (practice 3x)
- **Day 25-27**: Mock interviews with peers
- **Day 28-30**: Review [Common Questions](./interview-tips/questions.md) + final practice

## 🎯 Interview Checklist

### Before the Interview
- [ ] Reviewed [RADIO Framework](./interview-tips/approach.md)
- [ ] Practiced 3+ case studies
- [ ] Can draw system diagrams quickly
- [ ] Know capacity estimation formulas
- [ ] Understand trade-offs for each component

### During the Interview
- [ ] Clarify requirements (5-8 minutes)
- [ ] Start with simple design
- [ ] Add complexity gradually
- [ ] Explain trade-offs
- [ ] Handle follow-up questions

### Key Numbers to Remember
- **Latency**: L1 cache (1ns), RAM (100ns), SSD (100μs), Network (1ms)
- **Throughput**: Single server (1K QPS), Database (10K QPS), Cache (100K QPS)
- **Storage**: 1 user = 1KB, 1 post = 1KB, 1 image = 1MB, 1 video = 100MB

## 🔗 External Resources

### Books
- "Designing Data-Intensive Applications" by Martin Kleppmann
- "System Design Interview" by Alex Xu

### Websites
- [High Scalability](http://highscalability.com/)
- [AWS Architecture Center](https://aws.amazon.com/architecture/)
- [System Design Primer](https://github.com/donnemartin/system-design-primer)

### Practice Platforms
- Pramp, InterviewBit, LeetCode System Design

---

**💡 Pro Tip**: Focus on understanding concepts rather than memorizing solutions. Every system design interview is unique, but the fundamental principles remain the same.