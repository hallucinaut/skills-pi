---
name: caching
description: "Implement high-performance caching solutions with Redis and Memcached, including cache strategies, distributed caching, and performance optimization."
---

# Caching Skill

Implement high-performance caching solutions with Redis and Memcached for scalable applications.

## When to Use

Use this skill when the user wants to:
- Implement Redis or Memcached caching
- Reduce database load and improve performance
- Implement cache-aside, write-through, or write-behind patterns
- Set up distributed caching for microservices
- Implement session storage and management
- Cache API responses and database queries
- Implement rate limiting with Redis
- Set up cache invalidation strategies
- Implement real-time features with Redis pub/sub
- Create leaderboards and ranking systems
- Implement distributed locks and semaphores
- Set up cache warming and preloading

## Technology Stack

### Core Technologies

#### Redis
- **In-memory data structure store**
  - Strings, hashes, lists, sets, sorted sets
  - Pub/sub messaging
  - Streams for event sourcing
  - Geospatial indexes
  - HyperLogLog for counting
  - Bitmap operations

#### Memcached
- **Simple, high-performance caching**
  - Key-value storage
  - LRU eviction
  - Distributed architecture
  - Multi-threaded performance

### Client Libraries

#### Python
- **redis-py**: Official Redis client
- **aioredis**: Async Redis client
- **python-memcached**: Memcached client
- **pymemcache**: High-performance Memcached client
- **redis-om**: Object mapper for Redis

#### Node.js
- **ioredis**: Feature-rich Redis client
- **node-redis**: Official Redis client
- **memjs**: Memcached client

#### Java
- **Jedis**: Redis Java client
- **Lettuce**: Advanced Redis client
- **Redisson**: Redis framework

### Supporting Tools
- **Redis Sentinel**: High availability
- **Redis Cluster**: Horizontal scaling
- **RedisInsight**: GUI management tool
- **redis-cli**: Command-line interface
- **Twemproxy**: Proxy for Redis/Memcached

## Cache Strategies

### 1. Cache-Aside (Lazy Loading)

**Pattern:**
- Application checks cache first
- On cache miss, load from database
- Store result in cache for future requests

**Use Cases:**
- Read-heavy workloads
- Data that doesn't change frequently
- Queries with variable parameters

**Python Example:**
```python
import redis
from typing import Optional
import json

class CacheAside:
    def __init__(self, redis_client: redis.Redis):
        self.redis = redis_client
        self.default_ttl = 3600  # 1 hour

    def get_user(self, user_id: int) -> Optional[dict]:
        """Get user with cache-aside pattern."""
        cache_key = f"user:{user_id}"

        # Try cache first
        cached_data = self.redis.get(cache_key)
        if cached_data:
            return json.loads(cached_data)

        # Cache miss - load from database
        user = self._load_user_from_db(user_id)
        if user:
            # Store in cache
            self.redis.setex(
                cache_key,
                self.default_ttl,
                json.dumps(user)
            )

        return user

    def _load_user_from_db(self, user_id: int) -> Optional[dict]:
        """Load user from database."""
        # Database query implementation
        pass
```

### 2. Write-Through Cache

**Pattern:**
- Write to cache and database simultaneously
- Cache always in sync with database
- Slower writes, faster reads

**Use Cases:**
- Data consistency is critical
- Read-heavy workloads with some writes
- Data that must always be available

**Python Example:**
```python
class WriteThrough:
    def __init__(self, redis_client: redis.Redis, db_connection):
        self.redis = redis_client
        self.db = db_connection

    def update_user(self, user_id: int, data: dict) -> bool:
        """Update user with write-through pattern."""
        cache_key = f"user:{user_id}"

        try:
            # Write to database first
            self.db.update_user(user_id, data)

            # Update cache
            self.redis.setex(
                cache_key,
                3600,
                json.dumps(data)
            )

            return True
        except Exception as e:
            # Rollback if needed
            raise
```

### 3. Write-Behind (Write-Back)

**Pattern:**
- Write to cache immediately
- Asynchronously write to database
- Fastest writes, eventual consistency

**Use Cases:**
- High write throughput required
- Temporary data acceptable
- Analytics and metrics

**Python Example:**
```python
import asyncio
from collections import deque

class WriteBehind:
    def __init__(self, redis_client: redis.Redis, db_connection):
        self.redis = redis_client
        self.db = db_connection
        self.write_queue = deque()
        self.batch_size = 100
        self.flush_interval = 5  # seconds

    async def update_metric(self, metric_id: str, value: float):
        """Update metric with write-behind pattern."""
        cache_key = f"metric:{metric_id}"

        # Write to cache immediately
        self.redis.incrbyfloat(cache_key, value)

        # Queue for database write
        self.write_queue.append((metric_id, value))

        # Flush if batch size reached
        if len(self.write_queue) >= self.batch_size:
            await self._flush_to_db()

    async def _flush_to_db(self):
        """Flush queued writes to database."""
        if not self.write_queue:
            return

        batch = []
        while self.write_queue and len(batch) < self.batch_size:
            batch.append(self.write_queue.popleft())

        # Batch write to database
        await self.db.batch_update_metrics(batch)
```

### 4. Read-Through Cache

**Pattern:**
- Cache handles database loading
- Application only interacts with cache
- Cache library manages data loading

**Python Example:**
```python
class ReadThrough:
    def __init__(self, redis_client: redis.Redis):
        self.redis = redis_client

    def get_with_loader(self, key: str, loader_func, ttl: int = 3600):
        """Get data with automatic loading."""
        # Try cache
        cached = self.redis.get(key)
        if cached:
            return json.loads(cached)

        # Load using provided function
        data = loader_func()

        # Cache the result
        if data:
            self.redis.setex(key, ttl, json.dumps(data))

        return data
```

## Redis Implementation Patterns

### 1. Connection Management

**Python with Connection Pooling:**
```python
import redis
from redis.sentinel import Sentinel

class RedisManager:
    def __init__(self, config: dict):
        self.config = config
        self._pool = None
        self._client = None

    def get_client(self) -> redis.Redis:
        """Get Redis client with connection pooling."""
        if self._client is None:
            self._pool = redis.ConnectionPool(
                host=self.config.get('host', 'localhost'),
                port=self.config.get('port', 6379),
                db=self.config.get('db', 0),
                password=self.config.get('password'),
                max_connections=self.config.get('max_connections', 50),
                socket_timeout=self.config.get('timeout', 5),
                socket_connect_timeout=self.config.get('connect_timeout', 5),
                decode_responses=True
            )
            self._client = redis.Redis(connection_pool=self._pool)

        return self._client

    def get_sentinel_client(self, sentinel_hosts: list,
                           master_name: str) -> redis.Redis:
        """Get Redis client via Sentinel for high availability."""
        sentinel = Sentinel(
            sentinel_hosts,
            socket_timeout=0.5,
            sentinel_kwargs={'password': self.config.get('password')}
        )

        # Get master connection
        master = sentinel.master_for(
            master_name,
            socket_timeout=5,
            password=self.config.get('password')
        )

        return master

    def close(self):
        """Close connection pool."""
        if self._pool:
            self._pool.disconnect()
```

### 2. Cache Invalidation

**Invalidation Strategies:**
```python
class CacheInvalidation:
    def __init__(self, redis_client: redis.Redis):
        self.redis = redis_client

    def invalidate_by_key(self, key: str):
        """Delete specific cache key."""
        self.redis.delete(key)

    def invalidate_by_pattern(self, pattern: str):
        """Delete keys matching pattern."""
        cursor = 0
        while True:
            cursor, keys = self.redis.scan(cursor, match=pattern, count=100)
            if keys:
                self.redis.delete(*keys)
            if cursor == 0:
                break

    def invalidate_by_tags(self, tag: str):
        """Invalidate all keys with specific tag."""
        tag_key = f"tag:{tag}"
        keys = self.redis.smembers(tag_key)

        if keys:
            # Delete tagged keys
            self.redis.delete(*keys)
            # Delete tag set
            self.redis.delete(tag_key)

    def add_tag(self, key: str, tag: str, ttl: int = None):
        """Tag a cache key for group invalidation."""
        tag_key = f"tag:{tag}"
        self.redis.sadd(tag_key, key)

        if ttl:
            self.redis.expire(tag_key, ttl)
```

### 3. Distributed Locking

**Redis Distributed Lock:**
```python
import uuid
import time

class RedisLock:
    def __init__(self, redis_client: redis.Redis):
        self.redis = redis_client

    def acquire_lock(self, lock_name: str, timeout: int = 10,
                     ttl: int = 10) -> Optional[str]:
        """Acquire distributed lock."""
        identifier = str(uuid.uuid4())
        lock_key = f"lock:{lock_name}"
        end = time.time() + timeout

        while time.time() < end:
            # Try to acquire lock
            if self.redis.set(lock_key, identifier, nx=True, ex=ttl):
                return identifier

            # Wait before retry
            time.sleep(0.001)

        return None

    def release_lock(self, lock_name: str, identifier: str) -> bool:
        """Release distributed lock safely."""
        lock_key = f"lock:{lock_name}"

        # Use Lua script for atomic release
        lua_script = """
        if redis.call("get", KEYS[1]) == ARGV[1] then
            return redis.call("del", KEYS[1])
        else
            return 0
        end
        """

        return bool(self.redis.eval(lua_script, 1, lock_key, identifier))

    def __enter__(self):
        """Context manager support."""
        self.identifier = self.acquire_lock(self.lock_name)
        if not self.identifier:
            raise Exception("Could not acquire lock")
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        """Release lock on exit."""
        self.release_lock(self.lock_name, self.identifier)
```

### 4. Rate Limiting

**Token Bucket Rate Limiter:**
```python
from datetime import datetime

class RateLimiter:
    def __init__(self, redis_client: redis.Redis):
        self.redis = redis_client

    def is_allowed(self, user_id: str, max_requests: int = 100,
                   window_seconds: int = 60) -> bool:
        """Check if request is allowed (sliding window)."""
        key = f"rate_limit:{user_id}"
        now = time.time()
        window_start = now - window_seconds

        pipe = self.redis.pipeline()

        # Remove old entries
        pipe.zremrangebyscore(key, 0, window_start)

        # Count requests in window
        pipe.zcard(key)

        # Add current request
        pipe.zadd(key, {str(now): now})

        # Set expiry
        pipe.expire(key, window_seconds)

        results = pipe.execute()
        request_count = results[1]

        return request_count < max_requests

    def fixed_window_limiter(self, user_id: str, max_requests: int = 100,
                            window_seconds: int = 60) -> bool:
        """Fixed window rate limiter (simpler, less accurate)."""
        key = f"rate_limit:fixed:{user_id}"
        current_window = int(time.time() / window_seconds)
        window_key = f"{key}:{current_window}"

        pipe = self.redis.pipeline()
        pipe.incr(window_key)
        pipe.expire(window_key, window_seconds * 2)

        results = pipe.execute()
        request_count = results[0]

        return request_count <= max_requests
```

### 5. Session Management

**Redis Session Store:**
```python
class SessionStore:
    def __init__(self, redis_client: redis.Redis):
        self.redis = redis_client
        self.default_ttl = 3600  # 1 hour

    def create_session(self, user_id: str, data: dict,
                      ttl: int = None) -> str:
        """Create new session."""
        session_id = str(uuid.uuid4())
        session_key = f"session:{session_id}"

        session_data = {
            'user_id': user_id,
            'created_at': datetime.utcnow().isoformat(),
            **data
        }

        self.redis.setex(
            session_key,
            ttl or self.default_ttl,
            json.dumps(session_data)
        )

        # Index by user
        self.redis.sadd(f"user_sessions:{user_id}", session_id)

        return session_id

    def get_session(self, session_id: str) -> Optional[dict]:
        """Get session data."""
        session_key = f"session:{session_id}"
        data = self.redis.get(session_key)

        if data:
            return json.loads(data)
        return None

    def update_session(self, session_id: str, data: dict):
        """Update session data."""
        session = self.get_session(session_id)
        if session:
            session.update(data)
            session_key = f"session:{session_id}"
            ttl = self.redis.ttl(session_key)

            self.redis.setex(
                session_key,
                ttl if ttl > 0 else self.default_ttl,
                json.dumps(session)
            )

    def destroy_session(self, session_id: str):
        """Destroy session."""
        session = self.get_session(session_id)
        if session:
            session_key = f"session:{session_id}"
            user_id = session.get('user_id')

            self.redis.delete(session_key)
            if user_id:
                self.redis.srem(f"user_sessions:{user_id}", session_id)
```

### 6. Pub/Sub Messaging

**Redis Pub/Sub:**
```python
import threading

class PubSubManager:
    def __init__(self, redis_client: redis.Redis):
        self.redis = redis_client
        self.pubsub = self.redis.pubsub()
        self.listeners = {}

    def publish(self, channel: str, message: dict):
        """Publish message to channel."""
        self.redis.publish(channel, json.dumps(message))

    def subscribe(self, channel: str, callback):
        """Subscribe to channel with callback."""
        if channel not in self.listeners:
            self.pubsub.subscribe(channel)
            self.listeners[channel] = []

        self.listeners[channel].append(callback)

    def start_listening(self):
        """Start listening for messages in background thread."""
        def listen():
            for message in self.pubsub.listen():
                if message['type'] == 'message':
                    channel = message['channel'].decode('utf-8')
                    data = json.loads(message['data'])

                    for callback in self.listeners.get(channel, []):
                        try:
                            callback(data)
                        except Exception as e:
                            print(f"Error in callback: {e}")

        thread = threading.Thread(target=listen, daemon=True)
        thread.start()
```

## Memcached Implementation

### Basic Memcached Usage

**Python Example:**
```python
import pymemcache
from pymemcache.client.base import Client

class MemcachedManager:
    def __init__(self, host: str = 'localhost', port: int = 11211):
        self.client = Client(
            (host, port),
            serializer=self._serialize,
            deserializer=self._deserialize,
            connect_timeout=5,
            timeout=2
        )

    def _serialize(self, key, value):
        """Serialize value for storage."""
        if isinstance(value, str):
            return value.encode('utf-8'), 1
        return json.dumps(value).encode('utf-8'), 2

    def _deserialize(self, key, value, flags):
        """Deserialize value from storage."""
        if flags == 1:
            return value.decode('utf-8')
        if flags == 2:
            return json.loads(value.decode('utf-8'))
        return value

    def get(self, key: str):
        """Get value from cache."""
        return self.client.get(key)

    def set(self, key: str, value, ttl: int = 3600):
        """Set value in cache."""
        return self.client.set(key, value, expire=ttl)

    def delete(self, key: str):
        """Delete key from cache."""
        return self.client.delete(key)

    def increment(self, key: str, delta: int = 1):
        """Increment counter."""
        return self.client.incr(key, delta)

    def get_multi(self, keys: list):
        """Get multiple keys."""
        return self.client.get_many(keys)

    def set_multi(self, mapping: dict, ttl: int = 3600):
        """Set multiple keys."""
        return self.client.set_many(mapping, expire=ttl)
```

## Cache Performance Optimization

### 1. Cache Warming

**Preload Critical Data:**
```python
class CacheWarmer:
    def __init__(self, redis_client: redis.Redis):
        self.redis = redis_client

    async def warm_cache(self, data_loader):
        """Warm cache with frequently accessed data."""
        # Load hot data
        hot_items = await data_loader.get_popular_items(limit=1000)

        pipe = self.redis.pipeline()
        for item in hot_items:
            key = f"item:{item['id']}"
            pipe.setex(key, 3600, json.dumps(item))

        pipe.execute()

    def warm_on_startup(self):
        """Warm cache on application startup."""
        # Load configuration
        # Preload user sessions
        # Cache static content
        pass
```

### 2. Cache Compression

**Compress Large Values:**
```python
import gzip
import pickle

class CompressedCache:
    def __init__(self, redis_client: redis.Redis,
                 compression_threshold: int = 1024):
        self.redis = redis_client
        self.threshold = compression_threshold

    def set_compressed(self, key: str, value, ttl: int = 3600):
        """Set value with automatic compression."""
        serialized = pickle.dumps(value)

        if len(serialized) > self.threshold:
            # Compress large values
            compressed = gzip.compress(serialized)
            self.redis.setex(f"{key}:c", ttl, compressed)
        else:
            self.redis.setex(key, ttl, serialized)

    def get_compressed(self, key: str):
        """Get value with automatic decompression."""
        # Try compressed version first
        compressed = self.redis.get(f"{key}:c")
        if compressed:
            decompressed = gzip.decompress(compressed)
            return pickle.loads(decompressed)

        # Try uncompressed
        data = self.redis.get(key)
        if data:
            return pickle.loads(data)

        return None
```

### 3. Pipeline Batching

**Batch Operations for Performance:**
```python
class BatchCache:
    def __init__(self, redis_client: redis.Redis):
        self.redis = redis_client

    def batch_get(self, keys: list) -> dict:
        """Get multiple keys in single round-trip."""
        pipe = self.redis.pipeline()
        for key in keys:
            pipe.get(key)

        results = pipe.execute()

        return {
            key: json.loads(value) if value else None
            for key, value in zip(keys, results)
        }

    def batch_set(self, items: dict, ttl: int = 3600):
        """Set multiple keys in single round-trip."""
        pipe = self.redis.pipeline()
        for key, value in items.items():
            pipe.setex(key, ttl, json.dumps(value))

        pipe.execute()
```

## Monitoring and Debugging

### Redis Monitoring

**Performance Metrics:**
```python
class RedisMonitor:
    def __init__(self, redis_client: redis.Redis):
        self.redis = redis_client

    def get_stats(self) -> dict:
        """Get Redis statistics."""
        info = self.redis.info()

        return {
            'used_memory': info['used_memory_human'],
            'connected_clients': info['connected_clients'],
            'total_commands_processed': info['total_commands_processed'],
            'keyspace_hits': info['keyspace_hits'],
            'keyspace_misses': info['keyspace_misses'],
            'hit_rate': self._calculate_hit_rate(info),
            'evicted_keys': info.get('evicted_keys', 0),
            'expired_keys': info.get('expired_keys', 0)
        }

    def _calculate_hit_rate(self, info: dict) -> float:
        """Calculate cache hit rate."""
        hits = info.get('keyspace_hits', 0)
        misses = info.get('keyspace_misses', 0)
        total = hits + misses

        if total == 0:
            return 0.0

        return (hits / total) * 100

    def monitor_slow_log(self, count: int = 10):
        """Get slow queries."""
        return self.redis.slowlog_get(count)
```

## Best Practices

### 1. Key Naming Conventions
- Use descriptive, hierarchical names: `user:123:profile`
- Include version in keys: `v1:user:123`
- Use consistent separators (colons recommended)
- Keep keys short but meaningful
- Avoid special characters

### 2. TTL Management
- Always set expiration for cache keys
- Use appropriate TTL based on data volatility
- Implement cache warming for critical data
- Monitor eviction rates

### 3. Connection Pooling
- Always use connection pools
- Configure appropriate pool size
- Set reasonable timeouts
- Handle connection failures gracefully

### 4. Error Handling
- Implement fallback to database on cache failure
- Log cache errors separately
- Use circuit breakers for cache dependencies
- Gracefully degrade when cache is unavailable

### 5. Security
- Enable password authentication
- Use TLS/SSL for connections
- Implement access controls
- Isolate cache instances by environment
- Regularly update Redis/Memcached

### 6. Memory Management
- Set maxmemory limits
- Configure eviction policies (LRU, LFU)
- Monitor memory usage
- Use compression for large values
- Clean up expired keys

### 7. Data Consistency
- Use appropriate cache invalidation
- Implement version tagging
- Consider cache stampede prevention
- Use locks for critical operations

## Common Anti-Patterns to Avoid

1. **Caching everything**: Only cache frequently accessed data
2. **No TTL**: Always set expiration times
3. **Caching mutable data**: Be careful with frequently changing data
4. **Large values**: Compress or split large objects
5. **No monitoring**: Track hit rates and performance
6. **Single point of failure**: Use replication and clustering
7. **Ignoring cache misses**: Optimize for cache miss scenarios

## Deliverables

When implementing caching, ensure you deliver:

1. **Cache Configuration**
   - Connection settings and pools
   - Eviction policies and memory limits
   - Replication and clustering setup

2. **Cache Layer Implementation**
   - Cache strategy selection (cache-aside, write-through, etc.)
   - Key naming conventions
   - TTL strategies

3. **Integration Code**
   - Database integration
   - API/service integration
   - Error handling and fallbacks

4. **Monitoring Setup**
   - Performance metrics
   - Hit/miss rate tracking
   - Memory usage monitoring

5. **Documentation**
   - Cache architecture diagram
   - Key patterns and conventions
   - Troubleshooting guide

## Quality Checklist

Before completing a caching implementation, verify:

- [ ] Appropriate cache strategy selected and implemented
- [ ] Connection pooling configured properly
- [ ] TTL set for all cache keys
- [ ] Key naming conventions documented and consistent
- [ ] Error handling and fallback mechanisms in place
- [ ] Cache invalidation strategy implemented
- [ ] Memory limits and eviction policies configured
- [ ] Monitoring and metrics collection set up
- [ ] Security (authentication, encryption) enabled
- [ ] Performance testing completed
- [ ] Cache hit rate is acceptable (>80% for hot data)
- [ ] Documentation includes architecture and patterns
- [ ] High availability configured (Sentinel/Cluster)
- [ ] Backup and disaster recovery plan documented
