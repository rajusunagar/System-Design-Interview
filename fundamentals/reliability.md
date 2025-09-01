# Reliability

## Definition
Reliability is the probability that a system performs correctly during a specific time duration. A reliable system continues to work correctly even when things go wrong (hardware failures, software bugs, human errors).

## Key Concepts

### Mean Time Between Failures (MTBF)
- Average time between system failures
- Higher MTBF = More reliable system
- **Formula**: MTBF = Total Operating Time / Number of Failures

### Mean Time to Recovery (MTTR)
- Average time to restore service after failure
- Lower MTTR = Faster recovery
- **Formula**: MTTR = Total Downtime / Number of Failures

### Availability vs Reliability
- **Availability**: System is operational when needed
- **Reliability**: System performs correctly without failure
- A system can be available but unreliable (frequent restarts)
- A system can be reliable but not highly available (long recovery times)

## Reliability Patterns

### 1. Redundancy

#### Hardware Redundancy
```
Primary Server ← Load Balancer → Backup Server
      ↓                              ↓
  Primary DB ←→ Replication ←→ Backup DB
```

#### Software Redundancy
- Multiple instances of the same service
- Different implementations of the same functionality
- Diverse programming languages or frameworks

#### Data Redundancy
- Database replication (master-slave, master-master)
- Distributed storage with multiple copies
- Cross-region data backup

### 2. Fault Tolerance

#### Graceful Degradation
```python
def get_user_recommendations(user_id):
    try:
        # Try ML-based recommendations
        return ml_service.get_recommendations(user_id)
    except MLServiceException:
        try:
            # Fallback to collaborative filtering
            return collaborative_filter.get_recommendations(user_id)
        except CollaborativeFilterException:
            # Final fallback to popular items
            return get_popular_items()
```

#### Circuit Breaker Pattern
```python
class CircuitBreaker:
    def __init__(self, failure_threshold=5, timeout=60):
        self.failure_count = 0
        self.failure_threshold = failure_threshold
        self.timeout = timeout
        self.state = "CLOSED"  # CLOSED, OPEN, HALF_OPEN
        self.last_failure_time = 0
    
    def call(self, func, *args, **kwargs):
        if self.state == "OPEN":
            if time.time() - self.last_failure_time > self.timeout:
                self.state = "HALF_OPEN"
            else:
                raise Exception("Circuit breaker is OPEN")
        
        try:
            result = func(*args, **kwargs)
            if self.state == "HALF_OPEN":
                self.state = "CLOSED"
                self.failure_count = 0
            return result
        except Exception as e:
            self.failure_count += 1
            self.last_failure_time = time.time()
            if self.failure_count >= self.failure_threshold:
                self.state = "OPEN"
            raise e
```

### 3. Error Handling and Recovery

#### Retry Mechanisms
```python
import time
import random

def exponential_backoff_retry(func, max_retries=3, base_delay=1):
    for attempt in range(max_retries):
        try:
            return func()
        except Exception as e:
            if attempt == max_retries - 1:
                raise e
            
            # Exponential backoff with jitter
            delay = base_delay * (2 ** attempt) + random.uniform(0, 1)
            time.sleep(delay)
```

#### Dead Letter Queues
```python
class MessageProcessor:
    def __init__(self):
        self.max_retries = 3
        self.dead_letter_queue = DeadLetterQueue()
    
    def process_message(self, message):
        try:
            # Process the message
            business_logic(message)
        except Exception as e:
            if message.retry_count < self.max_retries:
                message.retry_count += 1
                # Retry with exponential backoff
                schedule_retry(message, delay=2 ** message.retry_count)
            else:
                # Send to dead letter queue for manual investigation
                self.dead_letter_queue.send(message, error=str(e))
```

## Reliability Techniques

### 1. Health Checks

#### Application Health Checks
```python
from flask import Flask, jsonify
import psutil
import redis

app = Flask(__name__)

@app.route('/health')
def health_check():
    health_status = {
        'status': 'healthy',
        'timestamp': time.time(),
        'checks': {}
    }
    
    # Database connectivity
    try:
        db.execute('SELECT 1')
        health_status['checks']['database'] = 'healthy'
    except Exception as e:
        health_status['checks']['database'] = f'unhealthy: {str(e)}'
        health_status['status'] = 'unhealthy'
    
    # Redis connectivity
    try:
        redis_client.ping()
        health_status['checks']['redis'] = 'healthy'
    except Exception as e:
        health_status['checks']['redis'] = f'unhealthy: {str(e)}'
        health_status['status'] = 'unhealthy'
    
    # System resources
    cpu_usage = psutil.cpu_percent()
    memory_usage = psutil.virtual_memory().percent
    
    if cpu_usage > 90 or memory_usage > 90:
        health_status['status'] = 'degraded'
    
    health_status['checks']['cpu_usage'] = f'{cpu_usage}%'
    health_status['checks']['memory_usage'] = f'{memory_usage}%'
    
    status_code = 200 if health_status['status'] == 'healthy' else 503
    return jsonify(health_status), status_code
```

#### Deep Health Checks
```python
@app.route('/health/deep')
def deep_health_check():
    checks = {}
    
    # Test critical business functionality
    try:
        # Simulate a typical user operation
        test_user = create_test_user()
        test_order = create_test_order(test_user.id)
        process_test_payment(test_order.id)
        cleanup_test_data(test_user.id, test_order.id)
        checks['end_to_end'] = 'healthy'
    except Exception as e:
        checks['end_to_end'] = f'unhealthy: {str(e)}'
    
    # Test external dependencies
    for service in ['payment_gateway', 'email_service', 'sms_service']:
        try:
            response = requests.get(f'{service_urls[service]}/health', timeout=5)
            checks[service] = 'healthy' if response.status_code == 200 else 'unhealthy'
        except Exception as e:
            checks[service] = f'unhealthy: {str(e)}'
    
    return jsonify(checks)
```

### 2. Monitoring and Alerting

#### Application Metrics
```python
from prometheus_client import Counter, Histogram, Gauge

# Define metrics
request_count = Counter('http_requests_total', 'Total HTTP requests', ['method', 'endpoint', 'status'])
request_duration = Histogram('http_request_duration_seconds', 'HTTP request duration')
active_connections = Gauge('active_connections', 'Number of active connections')
error_rate = Counter('errors_total', 'Total errors', ['error_type'])

def track_request(func):
    def wrapper(*args, **kwargs):
        start_time = time.time()
        try:
            result = func(*args, **kwargs)
            request_count.labels(method='GET', endpoint='/api/users', status='200').inc()
            return result
        except Exception as e:
            error_rate.labels(error_type=type(e).__name__).inc()
            request_count.labels(method='GET', endpoint='/api/users', status='500').inc()
            raise
        finally:
            request_duration.observe(time.time() - start_time)
    return wrapper
```

#### Custom Reliability Metrics
```python
class ReliabilityMetrics:
    def __init__(self):
        self.success_count = 0
        self.failure_count = 0
        self.start_time = time.time()
    
    def record_success(self):
        self.success_count += 1
    
    def record_failure(self):
        self.failure_count += 1
    
    def get_reliability(self):
        total_operations = self.success_count + self.failure_count
        if total_operations == 0:
            return 1.0
        return self.success_count / total_operations
    
    def get_mtbf(self):
        if self.failure_count == 0:
            return float('inf')
        uptime = time.time() - self.start_time
        return uptime / self.failure_count
```

### 3. Backup and Recovery

#### Database Backup Strategy
```python
class BackupManager:
    def __init__(self):
        self.s3_client = boto3.client('s3')
        self.backup_bucket = 'company-db-backups'
    
    def create_full_backup(self):
        timestamp = datetime.now().strftime('%Y%m%d_%H%M%S')
        backup_file = f'full_backup_{timestamp}.sql'
        
        # Create database dump
        subprocess.run([
            'pg_dump',
            '-h', 'localhost',
            '-U', 'postgres',
            '-d', 'production_db',
            '-f', backup_file
        ])
        
        # Upload to S3
        self.s3_client.upload_file(
            backup_file,
            self.backup_bucket,
            f'full_backups/{backup_file}'
        )
        
        # Clean up local file
        os.remove(backup_file)
        
        return f's3://{self.backup_bucket}/full_backups/{backup_file}'
    
    def create_incremental_backup(self):
        # Use WAL (Write-Ahead Logging) for incremental backups
        wal_files = self.get_new_wal_files()
        
        for wal_file in wal_files:
            self.s3_client.upload_file(
                wal_file,
                self.backup_bucket,
                f'wal_backups/{os.path.basename(wal_file)}'
            )
    
    def restore_from_backup(self, backup_path, target_time=None):
        if target_time:
            # Point-in-time recovery
            self.restore_point_in_time(backup_path, target_time)
        else:
            # Full restore
            self.restore_full_backup(backup_path)
```

#### Application State Backup
```python
class ApplicationStateManager:
    def __init__(self):
        self.redis_client = redis.Redis()
        self.state_backup_key = 'app_state_backup'
    
    def save_application_state(self):
        state = {
            'active_sessions': self.get_active_sessions(),
            'in_progress_transactions': self.get_pending_transactions(),
            'cache_state': self.get_critical_cache_data(),
            'timestamp': time.time()
        }
        
        # Save to Redis with expiration
        self.redis_client.setex(
            self.state_backup_key,
            3600,  # 1 hour expiration
            json.dumps(state)
        )
    
    def restore_application_state(self):
        state_data = self.redis_client.get(self.state_backup_key)
        if state_data:
            state = json.loads(state_data)
            self.restore_sessions(state['active_sessions'])
            self.restore_transactions(state['in_progress_transactions'])
            self.restore_cache(state['cache_state'])
```

## Reliability Testing

### Chaos Engineering
```python
import random
import time

class ChaosMonkey:
    def __init__(self, services):
        self.services = services
        self.enabled = True
    
    def random_failure(self):
        if not self.enabled:
            return
        
        service = random.choice(self.services)
        failure_type = random.choice(['kill_instance', 'network_delay', 'disk_full'])
        
        if failure_type == 'kill_instance':
            self.kill_random_instance(service)
        elif failure_type == 'network_delay':
            self.inject_network_delay(service)
        elif failure_type == 'disk_full':
            self.simulate_disk_full(service)
    
    def kill_random_instance(self, service):
        instances = self.get_service_instances(service)
        if len(instances) > 1:  # Don't kill the last instance
            victim = random.choice(instances)
            self.terminate_instance(victim)
            print(f"Chaos Monkey killed instance {victim} of service {service}")
    
    def inject_network_delay(self, service):
        delay_ms = random.randint(100, 1000)
        self.add_network_delay(service, delay_ms)
        print(f"Chaos Monkey added {delay_ms}ms network delay to {service}")
        
        # Remove delay after some time
        time.sleep(random.randint(30, 300))
        self.remove_network_delay(service)
```

### Fault Injection Testing
```python
class FaultInjector:
    def __init__(self):
        self.active_faults = {}
    
    def inject_database_failure(self, duration=60):
        """Simulate database connection failures"""
        fault_id = f"db_failure_{int(time.time())}"
        
        # Mock database to raise exceptions
        original_execute = database.execute
        def failing_execute(*args, **kwargs):
            raise DatabaseConnectionError("Simulated database failure")
        
        database.execute = failing_execute
        self.active_faults[fault_id] = {
            'type': 'database_failure',
            'start_time': time.time(),
            'duration': duration,
            'original_function': original_execute
        }
        
        # Schedule fault removal
        threading.Timer(duration, self.remove_fault, args=[fault_id]).start()
    
    def inject_memory_pressure(self, memory_mb=1000):
        """Simulate high memory usage"""
        # Allocate memory to simulate pressure
        memory_hog = bytearray(memory_mb * 1024 * 1024)
        
        fault_id = f"memory_pressure_{int(time.time())}"
        self.active_faults[fault_id] = {
            'type': 'memory_pressure',
            'memory_hog': memory_hog
        }
    
    def remove_fault(self, fault_id):
        if fault_id in self.active_faults:
            fault = self.active_faults[fault_id]
            
            if fault['type'] == 'database_failure':
                # Restore original database function
                database.execute = fault['original_function']
            elif fault['type'] == 'memory_pressure':
                # Release memory
                del fault['memory_hog']
            
            del self.active_faults[fault_id]
```

## Reliability Patterns in Distributed Systems

### 1. Bulkhead Pattern
```python
class BulkheadExecutor:
    def __init__(self):
        # Separate thread pools for different operations
        self.read_pool = ThreadPoolExecutor(max_workers=10)
        self.write_pool = ThreadPoolExecutor(max_workers=5)
        self.analytics_pool = ThreadPoolExecutor(max_workers=3)
    
    def execute_read(self, func, *args, **kwargs):
        return self.read_pool.submit(func, *args, **kwargs)
    
    def execute_write(self, func, *args, **kwargs):
        return self.write_pool.submit(func, *args, **kwargs)
    
    def execute_analytics(self, func, *args, **kwargs):
        return self.analytics_pool.submit(func, *args, **kwargs)
```

### 2. Timeout Pattern
```python
import signal

class TimeoutManager:
    def __init__(self, timeout_seconds):
        self.timeout_seconds = timeout_seconds
    
    def __enter__(self):
        signal.signal(signal.SIGALRM, self._timeout_handler)
        signal.alarm(self.timeout_seconds)
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        signal.alarm(0)  # Cancel the alarm
    
    def _timeout_handler(self, signum, frame):
        raise TimeoutError(f"Operation timed out after {self.timeout_seconds} seconds")

# Usage
try:
    with TimeoutManager(5):  # 5-second timeout
        result = slow_external_api_call()
except TimeoutError:
    result = get_cached_result()
```

### 3. Saga Pattern for Distributed Transactions
```python
class SagaOrchestrator:
    def __init__(self):
        self.steps = []
        self.compensations = []
    
    def add_step(self, action, compensation):
        self.steps.append(action)
        self.compensations.append(compensation)
    
    def execute(self):
        completed_steps = []
        
        try:
            for i, step in enumerate(self.steps):
                result = step()
                completed_steps.append(i)
        except Exception as e:
            # Compensate in reverse order
            for step_index in reversed(completed_steps):
                try:
                    self.compensations[step_index]()
                except Exception as comp_error:
                    # Log compensation failure
                    logger.error(f"Compensation failed for step {step_index}: {comp_error}")
            raise e

# Usage
saga = SagaOrchestrator()
saga.add_step(
    action=lambda: payment_service.charge_card(order.total),
    compensation=lambda: payment_service.refund_card(order.total)
)
saga.add_step(
    action=lambda: inventory_service.reserve_items(order.items),
    compensation=lambda: inventory_service.release_items(order.items)
)
saga.add_step(
    action=lambda: shipping_service.create_shipment(order),
    compensation=lambda: shipping_service.cancel_shipment(order.id)
)

saga.execute()
```

## Reliability Metrics and SLAs

### Service Level Objectives (SLOs)
```python
class SLOMonitor:
    def __init__(self):
        self.slos = {
            'availability': 0.999,  # 99.9% uptime
            'latency_p99': 200,     # 99th percentile < 200ms
            'error_rate': 0.001     # < 0.1% error rate
        }
        self.metrics = {
            'total_requests': 0,
            'successful_requests': 0,
            'response_times': [],
            'uptime_start': time.time()
        }
    
    def record_request(self, response_time, success):
        self.metrics['total_requests'] += 1
        self.metrics['response_times'].append(response_time)
        
        if success:
            self.metrics['successful_requests'] += 1
    
    def check_slo_compliance(self):
        # Calculate current metrics
        error_rate = 1 - (self.metrics['successful_requests'] / self.metrics['total_requests'])
        p99_latency = np.percentile(self.metrics['response_times'], 99)
        
        # Check compliance
        compliance = {
            'error_rate': error_rate <= self.slos['error_rate'],
            'latency_p99': p99_latency <= self.slos['latency_p99']
        }
        
        return compliance
```

### Error Budget Management
```python
class ErrorBudgetManager:
    def __init__(self, slo_target=0.999, window_days=30):
        self.slo_target = slo_target
        self.window_days = window_days
        self.error_budget = 1 - slo_target  # 0.1% for 99.9% SLO
    
    def calculate_error_budget_consumption(self):
        # Get error rate over the window period
        end_time = time.time()
        start_time = end_time - (self.window_days * 24 * 3600)
        
        total_requests = self.get_request_count(start_time, end_time)
        failed_requests = self.get_error_count(start_time, end_time)
        
        if total_requests == 0:
            return 0
        
        actual_error_rate = failed_requests / total_requests
        budget_consumption = actual_error_rate / self.error_budget
        
        return min(budget_consumption, 1.0)  # Cap at 100%
    
    def is_error_budget_exhausted(self):
        return self.calculate_error_budget_consumption() >= 1.0
```

## Best Practices for Reliability

### 1. Design for Failure
- Assume components will fail
- Plan for graceful degradation
- Implement circuit breakers and timeouts
- Use bulkhead patterns to isolate failures

### 2. Monitoring and Observability
- Implement comprehensive health checks
- Monitor key reliability metrics
- Set up alerting for SLO violations
- Use distributed tracing for complex systems

### 3. Testing for Reliability
- Conduct chaos engineering experiments
- Perform disaster recovery drills
- Test failure scenarios regularly
- Validate backup and recovery procedures

### 4. Operational Excellence
- Automate deployment and rollback procedures
- Implement canary deployments
- Maintain runbooks for incident response
- Conduct post-incident reviews

## Interview Questions

1. **Reliability vs Availability**: What's the difference between reliability and availability?
2. **Failure Handling**: How do you design a system to handle component failures?
3. **Monitoring**: What metrics would you track to measure system reliability?
4. **Recovery**: How do you implement disaster recovery for a distributed system?
5. **Testing**: How do you test the reliability of a system?

## Real-World Examples

### Netflix
- **Chaos Monkey**: Randomly terminates instances in production
- **Circuit Breakers**: Hystrix library for fault tolerance
- **Bulkhead Pattern**: Separate thread pools for different operations

### Amazon
- **Cell-based Architecture**: Isolate failures to small groups of customers
- **Automated Recovery**: Self-healing systems that recover from failures
- **Redundancy**: Multiple availability zones and regions

### Google
- **SRE Practices**: Site Reliability Engineering methodology
- **Error Budgets**: Balance reliability with feature velocity
- **Chaos Engineering**: DiRT (Disaster Recovery Testing) exercises

Reliability is fundamental to building systems that users can depend on. It requires careful planning, robust design patterns, comprehensive testing, and continuous monitoring to achieve and maintain high levels of system reliability.