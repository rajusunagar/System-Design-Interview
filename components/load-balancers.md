# Load Balancers

## Definition
Load balancers distribute incoming network traffic across multiple servers to ensure no single server bears too much demand.

## Types of Load Balancers

### Layer 4 (Transport Layer)
- Routes based on IP and port
- **Pros**: Fast, simple, protocol agnostic
- **Cons**: No application awareness
- **Examples**: AWS NLB, HAProxy

### Layer 7 (Application Layer)
- Routes based on application data (HTTP headers, URLs)
- **Pros**: Intelligent routing, SSL termination
- **Cons**: Higher latency, more complex
- **Examples**: AWS ALB, Nginx, Cloudflare

## Load Balancing Algorithms

### Round Robin
```
Request 1 → Server A
Request 2 → Server B  
Request 3 → Server C
Request 4 → Server A (cycle repeats)
```
- **Pros**: Simple, fair distribution
- **Cons**: Doesn't consider server capacity

### Weighted Round Robin
- Assign weights based on server capacity
- Higher weight = more requests
- **Use Case**: Servers with different specifications

### Least Connections
- Route to server with fewest active connections
- **Pros**: Better for long-lived connections
- **Cons**: Requires connection tracking

### Least Response Time
- Route to server with fastest response time
- **Pros**: Optimizes user experience
- **Cons**: Complex to implement

### IP Hash
- Hash client IP to determine server
- **Pros**: Session affinity without sticky sessions
- **Cons**: Uneven distribution possible

### Geographic
- Route based on client location
- **Pros**: Reduced latency
- **Cons**: Complex setup

## Health Checks

### Active Health Checks
```
Load Balancer → Server: Health check request
Server → Load Balancer: Health status response
```

### Passive Health Checks
- Monitor actual traffic responses
- Remove servers that return errors
- **Advantage**: No additional overhead

### Health Check Parameters
- **Interval**: How often to check (e.g., 30 seconds)
- **Timeout**: Max wait time for response
- **Threshold**: Consecutive failures before marking unhealthy

## High Availability Patterns

### Active-Passive
```
Primary LB (Active) ← Client Traffic
Secondary LB (Standby) ← Failover
```

### Active-Active
```
LB1 (Active) ← 50% Traffic
LB2 (Active) ← 50% Traffic
```

### DNS Load Balancing
```
DNS Server returns multiple IP addresses
Client connects to one of them
```

## Session Management

### Sticky Sessions (Session Affinity)
- Route user to same server
- **Pros**: Simple session management
- **Cons**: Uneven load, server failure issues

### Session Replication
- Replicate session data across servers
- **Pros**: No single point of failure
- **Cons**: Network overhead

### External Session Store
```
Client → Load Balancer → Server
Server ↔ Redis/Database (Session Store)
```
- **Pros**: Stateless servers, scalable
- **Cons**: Additional complexity

## SSL/TLS Handling

### SSL Termination
- Load balancer handles SSL encryption/decryption
- **Pros**: Reduces server CPU load
- **Cons**: Internal traffic unencrypted

### SSL Pass-through
- Servers handle SSL directly
- **Pros**: End-to-end encryption
- **Cons**: Higher server CPU usage

### SSL Bridging
- Decrypt at load balancer, re-encrypt to servers
- **Pros**: Load balancer can inspect traffic
- **Cons**: Higher complexity

## Popular Load Balancers

### Hardware
- **F5 BIG-IP**: Enterprise-grade
- **Citrix NetScaler**: Application delivery
- **A10 Networks**: High performance

### Software
- **Nginx**: Web server + load balancer
- **HAProxy**: High availability proxy
- **Apache HTTP Server**: mod_proxy_balancer

### Cloud-based
- **AWS ALB/NLB**: Application/Network load balancer
- **Google Cloud Load Balancing**: Global load balancing
- **Azure Load Balancer**: Layer 4 load balancing
- **Cloudflare**: Global CDN + load balancing

## Best Practices

1. **Monitor Health**: Implement comprehensive health checks
2. **Plan Capacity**: Size for peak load + buffer
3. **Geographic Distribution**: Use multiple regions
4. **Graceful Degradation**: Handle server failures elegantly
5. **Security**: Implement DDoS protection
6. **Logging**: Monitor traffic patterns and errors

## Interview Questions

1. How would you design a load balancer for a global application?
2. What's the difference between Layer 4 and Layer 7 load balancing?
3. How do you handle session management in a load-balanced environment?
4. Design a load balancing strategy for a video streaming service
5. How do you ensure load balancer high availability?

## Real-World Examples

### Netflix
- Uses Zuul (API Gateway) for load balancing
- Ribbon for client-side load balancing
- Eureka for service discovery

### Amazon
- ELB for distributing traffic
- Route 53 for DNS-based load balancing
- CloudFront for global content delivery

### Google
- Global Load Balancer with Anycast
- Maglev for consistent hashing
- Andromeda for network virtualization