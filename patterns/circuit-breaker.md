# Circuit Breaker Pattern

## Definition
The Circuit Breaker pattern prevents an application from repeatedly trying to execute an operation that's likely to fail, allowing it to continue without waiting for the fault to be fixed or wasting CPU cycles.

## Problem Statement
In distributed systems, services often depend on external resources (databases, APIs, other services). When these dependencies fail or become slow, the calling service can:
- Waste resources waiting for timeouts
- Cascade failures throughout the system
- Degrade user experience with slow responses

## Circuit Breaker States

### 1. Closed State (Normal Operation)
```
Client → Circuit Breaker → Service
         (Passes through)
```
- All requests pass through to the service
- Monitor failure rate and response times
- Count consecutive failures

### 2. Open State (Failing Fast)
```
Client → Circuit Breaker ✗ Service
         (Fails immediately)
```
- All requests fail immediately without calling the service
- Return cached data or default response
- Start timeout timer for recovery attempt

### 3. Half-Open State (Testing Recovery)
```
Client → Circuit Breaker → Service
         (Limited requests)
```
- Allow limited number of test requests
- If requests succeed, transition to Closed
- If requests fail, return to Open state

## Implementation

### Basic Circuit Breaker
```python
import time
from enum import Enum
from threading import Lock

class CircuitState(Enum):
    CLOSED = "CLOSED"
    OPEN = "OPEN"
    HALF_OPEN = "HALF_OPEN"

class CircuitBreaker:
    def __init__(self, 
                 failure_threshold=5,
                 recovery_timeout=60,
                 expected_exception=Exception):
        self.failure_threshold = failure_threshold
        self.recovery_timeout = recovery_timeout
        self.expected_exception = expected_exception
        
        self.failure_count = 0
        self.last_failure_time = None
        self.state = CircuitState.CLOSED
        self.lock = Lock()
    
    def call(self, func, *args, **kwargs):
        with self.lock:
            if self.state == CircuitState.OPEN:
                if self._should_attempt_reset():
                    self.state = CircuitState.HALF_OPEN
                else:
                    raise Exception("Circuit breaker is OPEN")
            
            try:
                result = func(*args, **kwargs)
                self._on_success()
                return result
            except self.expected_exception as e:
                self._on_failure()
                raise e
    
    def _should_attempt_reset(self):
        return (time.time() - self.last_failure_time) >= self.recovery_timeout
    
    def _on_success(self):
        self.failure_count = 0
        self.state = CircuitState.CLOSED
    
    def _on_failure(self):
        self.failure_count += 1
        self.last_failure_time = time.time()
        
        if self.failure_count >= self.failure_threshold:
            self.state = CircuitState.OPEN
```

### Advanced Circuit Breaker with Metrics
```python
import time
import threading
from collections import deque
from dataclasses import dataclass
from typing import Callable, Any, Optional

@dataclass
class CircuitBreakerConfig:
    failure_threshold: int = 5
    success_threshold: int = 3
    timeout: int = 60
    window_size: int = 100
    minimum_requests: int = 10

class AdvancedCircuitBreaker:
    def __init__(self, config: CircuitBreakerConfig):
        self.config = config
        self.state = CircuitState.CLOSED
        self.failure_count = 0
        self.success_count = 0
        self.last_failure_time = 0
        self.request_log = deque(maxlen=config.window_size)
        self.lock = threading.RLock()
        
    def call(self, func: Callable, *args, **kwargs) -> Any:
        with self.lock:
            if self.state == CircuitState.OPEN:
                if not self._should_attempt_reset():
                    raise CircuitBreakerOpenException("Circuit breaker is OPEN")
                self.state = CircuitState.HALF_OPEN
                self.success_count = 0
            
            if self.state == CircuitState.HALF_OPEN:
                if self.success_count >= self.config.success_threshold:
                    self._reset()
            
            return self._execute_call(func, *args, **kwargs)
    
    def _execute_call(self, func: Callable, *args, **kwargs) -> Any:
        start_time = time.time()
        try:
            result = func(*args, **kwargs)
            self._record_success(time.time() - start_time)
            return result
        except Exception as e:
            self._record_failure(time.time() - start_time)
            raise e
    
    def _record_success(self, response_time: float):
        self.request_log.append({
            'timestamp': time.time(),
            'success': True,
            'response_time': response_time
        })
        
        if self.state == CircuitState.HALF_OPEN:
            self.success_count += 1
            if self.success_count >= self.config.success_threshold:
                self._reset()
        elif self.state == CircuitState.CLOSED:
            self.failure_count = 0
    
    def _record_failure(self, response_time: float):
        self.request_log.append({
            'timestamp': time.time(),
            'success': False,
            'response_time': response_time
        })
        
        self.failure_count += 1
        self.last_failure_time = time.time()
        
        if self._should_trip():
            self.state = CircuitState.OPEN
    
    def _should_trip(self) -> bool:
        if len(self.request_log) < self.config.minimum_requests:
            return False
        
        recent_failures = sum(1 for req in self.request_log if not req['success'])
        failure_rate = recent_failures / len(self.request_log)
        
        return failure_rate >= (self.config.failure_threshold / 100)
    
    def _should_attempt_reset(self) -> bool:
        return (time.time() - self.last_failure_time) >= self.config.timeout
    
    def _reset(self):
        self.state = CircuitState.CLOSED
        self.failure_count = 0
        self.success_count = 0
        self.request_log.clear()
    
    def get_metrics(self) -> dict:
        with self.lock:
            total_requests = len(self.request_log)
            if total_requests == 0:
                return {
                    'state': self.state.value,
                    'total_requests': 0,
                    'success_rate': 0,
                    'average_response_time': 0
                }
            
            successful_requests = sum(1 for req in self.request_log if req['success'])
            success_rate = successful_requests / total_requests
            avg_response_time = sum(req['response_time'] for req in self.request_log) / total_requests
            
            return {
                'state': self.state.value,
                'total_requests': total_requests,
                'success_rate': success_rate,
                'average_response_time': avg_response_time,
                'failure_count': self.failure_count
            }

class CircuitBreakerOpenException(Exception):
    pass
```

## Usage Examples

### Database Connection
```python
# Database service with circuit breaker
class DatabaseService:
    def __init__(self):
        self.circuit_breaker = CircuitBreaker(
            failure_threshold=3,
            recovery_timeout=30
        )
        self.db_connection = DatabaseConnection()
    
    def get_user(self, user_id):
        try:
            return self.circuit_breaker.call(
                self._fetch_user_from_db, 
                user_id
            )
        except CircuitBreakerOpenException:
            # Return cached data or default response
            return self._get_user_from_cache(user_id)
    
    def _fetch_user_from_db(self, user_id):
        return self.db_connection.query(
            "SELECT * FROM users WHERE id = %s", 
            user_id
        )
    
    def _get_user_from_cache(self, user_id):
        # Fallback to cache or return default user
        return cache.get(f"user:{user_id}") or default_user()
```

### HTTP API Calls
```python
import requests

class ExternalAPIService:
    def __init__(self):
        self.circuit_breaker = CircuitBreaker(
            failure_threshold=5,
            recovery_timeout=60,
            expected_exception=(requests.RequestException, requests.Timeout)
        )
    
    def get_weather_data(self, city):
        try:
            return self.circuit_breaker.call(
                self._call_weather_api,
                city
            )
        except CircuitBreakerOpenException:
            return self._get_cached_weather(city)
    
    def _call_weather_api(self, city):
        response = requests.get(
            f"https://api.weather.com/v1/weather/{city}",
            timeout=5
        )
        response.raise_for_status()
        return response.json()
    
    def _get_cached_weather(self, city):
        return {
            "city": city,
            "temperature": "N/A",
            "status": "Service temporarily unavailable"
        }
```

### Microservices Communication
```python
class UserService:
    def __init__(self):
        self.order_service_cb = CircuitBreaker(failure_threshold=3)
        self.payment_service_cb = CircuitBreaker(failure_threshold=2)
    
    def get_user_profile(self, user_id):
        user_data = self._get_user_basic_info(user_id)
        
        # Try to get order history
        try:
            orders = self.order_service_cb.call(
                self._get_user_orders, 
                user_id
            )
            user_data['orders'] = orders
        except CircuitBreakerOpenException:
            user_data['orders'] = []
            user_data['orders_unavailable'] = True
        
        # Try to get payment methods
        try:
            payments = self.payment_service_cb.call(
                self._get_payment_methods, 
                user_id
            )
            user_data['payment_methods'] = payments
        except CircuitBreakerOpenException:
            user_data['payment_methods'] = []
            user_data['payments_unavailable'] = True
        
        return user_data
```

## Circuit Breaker with Bulkhead Pattern
```python
class ServiceClient:
    def __init__(self):
        # Separate circuit breakers for different operations
        self.read_circuit_breaker = CircuitBreaker(
            failure_threshold=5,
            recovery_timeout=30
        )
        self.write_circuit_breaker = CircuitBreaker(
            failure_threshold=2,  # More sensitive for writes
            recovery_timeout=60
        )
    
    def read_data(self, query):
        return self.read_circuit_breaker.call(
            self._execute_read_query, 
            query
        )
    
    def write_data(self, data):
        return self.write_circuit_breaker.call(
            self._execute_write_query, 
            data
        )
```

## Monitoring and Alerting

### Metrics Collection
```python
class CircuitBreakerMonitor:
    def __init__(self, circuit_breaker, service_name):
        self.circuit_breaker = circuit_breaker
        self.service_name = service_name
        self.metrics_client = MetricsClient()
    
    def collect_metrics(self):
        metrics = self.circuit_breaker.get_metrics()
        
        # Send metrics to monitoring system
        self.metrics_client.gauge(
            f'circuit_breaker.{self.service_name}.state',
            1 if metrics['state'] == 'OPEN' else 0
        )
        
        self.metrics_client.gauge(
            f'circuit_breaker.{self.service_name}.success_rate',
            metrics['success_rate']
        )
        
        self.metrics_client.gauge(
            f'circuit_breaker.{self.service_name}.response_time',
            metrics['average_response_time']
        )
        
        # Alert if circuit breaker is open
        if metrics['state'] == 'OPEN':
            self._send_alert(f"Circuit breaker for {self.service_name} is OPEN")
    
    def _send_alert(self, message):
        # Send alert to monitoring system
        alert_client.send_alert(
            severity='WARNING',
            message=message,
            service=self.service_name
        )
```

### Dashboard Visualization
```python
class CircuitBreakerDashboard:
    def get_dashboard_data(self):
        return {
            'circuit_breakers': [
                {
                    'name': 'database_service',
                    'state': 'CLOSED',
                    'success_rate': 0.95,
                    'failure_count': 2,
                    'last_failure': '2024-01-01T10:30:00Z'
                },
                {
                    'name': 'payment_service',
                    'state': 'OPEN',
                    'success_rate': 0.20,
                    'failure_count': 8,
                    'last_failure': '2024-01-01T11:45:00Z'
                }
            ]
        }
```

## Configuration Strategies

### Environment-based Configuration
```python
import os

class CircuitBreakerFactory:
    @staticmethod
    def create_for_service(service_name):
        config = {
            'database': {
                'failure_threshold': int(os.getenv('DB_CB_FAILURE_THRESHOLD', 5)),
                'recovery_timeout': int(os.getenv('DB_CB_RECOVERY_TIMEOUT', 60))
            },
            'payment': {
                'failure_threshold': int(os.getenv('PAYMENT_CB_FAILURE_THRESHOLD', 2)),
                'recovery_timeout': int(os.getenv('PAYMENT_CB_RECOVERY_TIMEOUT', 120))
            }
        }
        
        service_config = config.get(service_name, config['database'])
        return CircuitBreaker(**service_config)
```

### Dynamic Configuration
```python
class DynamicCircuitBreaker(CircuitBreaker):
    def __init__(self, config_service, service_name):
        self.config_service = config_service
        self.service_name = service_name
        
        # Initialize with default config
        config = self._load_config()
        super().__init__(**config)
        
        # Periodically update configuration
        self._schedule_config_refresh()
    
    def _load_config(self):
        return self.config_service.get_circuit_breaker_config(
            self.service_name
        )
    
    def _schedule_config_refresh(self):
        # Refresh configuration every 5 minutes
        threading.Timer(300, self._refresh_config).start()
    
    def _refresh_config(self):
        new_config = self._load_config()
        
        # Update configuration without disrupting current state
        self.failure_threshold = new_config['failure_threshold']
        self.recovery_timeout = new_config['recovery_timeout']
        
        self._schedule_config_refresh()
```

## Best Practices

### 1. Appropriate Thresholds
```python
# Different thresholds for different service types
circuit_breakers = {
    'critical_service': CircuitBreaker(failure_threshold=2, recovery_timeout=30),
    'non_critical_service': CircuitBreaker(failure_threshold=10, recovery_timeout=60),
    'external_api': CircuitBreaker(failure_threshold=5, recovery_timeout=120)
}
```

### 2. Graceful Degradation
```python
def get_user_recommendations(user_id):
    try:
        return recommendation_circuit_breaker.call(
            ml_service.get_recommendations,
            user_id
        )
    except CircuitBreakerOpenException:
        # Fallback to simple recommendations
        return get_popular_items()
```

### 3. Proper Exception Handling
```python
class SmartCircuitBreaker(CircuitBreaker):
    def __init__(self, **kwargs):
        # Only count specific exceptions as failures
        self.failure_exceptions = (
            requests.ConnectionError,
            requests.Timeout,
            DatabaseConnectionError
        )
        super().__init__(**kwargs)
    
    def _is_failure(self, exception):
        return isinstance(exception, self.failure_exceptions)
```

## Testing Circuit Breakers

### Unit Tests
```python
import unittest
from unittest.mock import Mock

class TestCircuitBreaker(unittest.TestCase):
    def setUp(self):
        self.circuit_breaker = CircuitBreaker(
            failure_threshold=3,
            recovery_timeout=1
        )
        self.mock_service = Mock()
    
    def test_closed_state_passes_through(self):
        self.mock_service.return_value = "success"
        result = self.circuit_breaker.call(self.mock_service)
        self.assertEqual(result, "success")
        self.assertEqual(self.circuit_breaker.state, CircuitState.CLOSED)
    
    def test_opens_after_threshold_failures(self):
        self.mock_service.side_effect = Exception("Service error")
        
        # Trigger failures up to threshold
        for _ in range(3):
            with self.assertRaises(Exception):
                self.circuit_breaker.call(self.mock_service)
        
        # Circuit breaker should now be open
        self.assertEqual(self.circuit_breaker.state, CircuitState.OPEN)
    
    def test_fails_fast_when_open(self):
        # Force circuit breaker to open state
        self.circuit_breaker.state = CircuitState.OPEN
        self.circuit_breaker.last_failure_time = time.time()
        
        with self.assertRaises(Exception):
            self.circuit_breaker.call(self.mock_service)
        
        # Service should not be called
        self.mock_service.assert_not_called()
```

### Integration Tests
```python
class TestCircuitBreakerIntegration(unittest.TestCase):
    def test_database_circuit_breaker(self):
        db_service = DatabaseService()
        
        # Simulate database failures
        with patch.object(db_service.db_connection, 'query') as mock_query:
            mock_query.side_effect = DatabaseError("Connection failed")
            
            # Should eventually open circuit breaker
            for _ in range(5):
                try:
                    db_service.get_user(123)
                except:
                    pass
            
            # Verify fallback behavior
            result = db_service.get_user(123)
            self.assertIsNotNone(result)  # Should return cached/default data
```

## Real-World Examples

### Netflix Hystrix
- **Language**: Java
- **Features**: Circuit breaker, bulkhead, timeout
- **Monitoring**: Real-time dashboard
- **Fallbacks**: Graceful degradation

### Resilience4j
- **Language**: Java
- **Features**: Circuit breaker, retry, rate limiter
- **Integration**: Spring Boot, Micronaut
- **Metrics**: Micrometer integration

### Polly (.NET)
- **Language**: C#
- **Features**: Circuit breaker, retry, timeout
- **Policies**: Composable resilience patterns
- **Async Support**: Full async/await support

## Interview Questions

1. **When to use**: When would you implement a circuit breaker pattern?
2. **State transitions**: Explain the different states and transitions
3. **Configuration**: How do you determine appropriate thresholds?
4. **Monitoring**: What metrics would you track for circuit breakers?
5. **Testing**: How do you test circuit breaker behavior?
6. **Alternatives**: What are alternatives to circuit breaker pattern?

The Circuit Breaker pattern is essential for building resilient distributed systems that can handle failures gracefully and prevent cascade failures from bringing down entire systems.