---
name: performance-optimization
description: "Optimize application performance, reduce latency, and improve resource usage. Use when profiling code, fixing bottlenecks, or optimizing queries and algorithms."
---

# Performance Optimization Skill

Optimize application performance, reduce latency, and improve resource usage.

## When to Use

Use this skill when the user wants to:
- Analyze application performance
- Identify bottlenecks and slow code
- Optimize database queries
- Reduce memory usage
- Improve rendering performance
- Minimize load times
- Optimize network requests
- Profile and measure performance

## Performance Metrics

### Core Metrics
- **Latency**: Response time (ms)
- **Throughput**: Requests per second
- **Memory usage**: Heap size, allocation
- **CPU usage**: CPU time, load
- **Time to First Byte (TTFB)**: Server response time
- **Time to Interactive (TTI)**: Time to become interactive
- **First Contentful Paint (FCP)**: Time to first paint
- **Largest Contentful Paint (LCP)**: Largest element paint time

### Browser Metrics
- **FCP**: First Contentful Paint
- **LCP**: Largest Contentful Paint
- **FID**: First Input Delay
- **CLS**: Cumulative Layout Shift
- **TTI**: Time to Interactive
- **INP**: Interaction to Next Paint

## Optimization Strategies

### Code Optimization

#### Algorithm Complexity
```javascript
// O(n²) → O(n) or O(log n)
// Bad: Nested loops
for (let i = 0; i < items.length; i++) {
  for (let j = 0; j < items.length; j++) {
    process(items[i], items[j]);
  }
}

// Good: Use Set for O(1) lookups
const uniqueItems = new Set(items);
for (const item of items) {
  if (uniqueItems.has(item)) {
    process(item);
  }
}
```

#### Caching
```javascript
// In-memory cache
const cache = new Map();

function getExpensiveData(key) {
  if (cache.has(key)) {
    return cache.get(key);
  }
  const data = expensiveOperation();
  cache.set(key, data);
  return data;
}

// Cache expiration
setTimeout(() => cache.delete(key), 5 * 60 * 1000); // 5 minutes
```

#### Lazy Loading
```javascript
// Lazy initialization
let heavyModule = null;

function getHeavyModule() {
  if (!heavyModule) {
    heavyModule = import('./heavy-module.js');
  }
  return heavyModule;
}
```

#### Debouncing/Throttling
```javascript
// Debounce (execute once after inactivity)
function debounce(func, wait) {
  let timeout;
  return function() {
    clearTimeout(timeout);
    timeout = setTimeout(() => func.apply(this, arguments), wait);
  };
}

// Throttle (execute at most once per period)
function throttle(func, limit) {
  let inThrottle;
  return function() {
    if (!inThrottle) {
      func.apply(this, arguments);
      inThrottle = true;
      setTimeout(() => inThrottle = false, limit);
    }
  };
}
```

### Database Optimization

#### Query Optimization
```sql
-- Use indexes
CREATE INDEX idx_user_email ON users(email);

-- Avoid SELECT *
SELECT id, name, email FROM users WHERE email = ?;

-- Use JOIN instead of subqueries
SELECT * FROM users u JOIN orders o ON u.id = o.user_id WHERE o.status = 'paid';

-- Limit result size
SELECT * FROM users LIMIT 100;
```

#### N+1 Query Problem
```javascript
// Bad: N+1 queries
users.forEach(user => {
  const orders = db.getOrders(user.id); // N queries
});

// Good: Single query
const usersWithOrders = db.getUsersWithOrders();
```

### Frontend Optimization

#### Rendering Optimization
```javascript
// Virtual DOM (React)
// React only re-renders changed components

// Memoization
const expensiveComponent = React.memo(expensiveComponent);

// Code splitting
const LazyComponent = React.lazy(() => import('./LazyComponent'));
```

#### Asset Optimization
```javascript
// Image optimization
- Use modern formats (WebP, AVIF)
- Responsive images
- Lazy loading images
- Compression (gzip, Brotli)
- Minify HTML/CSS/JS
```

#### Bundle Optimization
```javascript
// Tree shaking
import { util, helper } from './utils.js'; // Only import used exports

// Lazy loading routes
const Dashboard = React.lazy(() => import('./Dashboard'));
```

### Network Optimization

#### Request Optimization
```javascript
// Fetch with cache
fetch(url, {
  headers: {
    'Cache-Control': 'max-age=3600' // Cache for 1 hour
  }
});

// Parallel requests
Promise.all([
  fetch('/api/user'),
  fetch('/api/orders'),
  fetch('/api/settings')
]);
```

#### Compression
```javascript
// Gzip compression
app.use(compression());

// Use modern compression
npm install compression
```

### Memory Optimization

#### Memory Leaks
```javascript
// Remove event listeners
component.removeEventListener('click', handleClick);

// Clear timers
clearInterval(timer);
clearTimeout(timeout);

// Null references
component = null;
```

#### Object Pooling
```javascript
// Reuse objects instead of creating new ones
class ObjectPool {
  constructor(createFn, resetFn, initialSize = 10) {
    this.pool = [];
    this.createFn = createFn;
    this.resetFn = resetFn;

    for (let i = 0; i < initialSize; i++) {
      this.pool.push(createFn());
    }
  }

  get() {
    return this.pool.length > 0 ? this.pool.pop() : this.createFn();
  }

  release(obj) {
    this.resetFn(obj);
    this.pool.push(obj);
  }
}
```

## Performance Profiling

### Browser Profiling
```javascript
// Chrome DevTools
1. Open DevTools (F12)
2. Performance tab
3. Record, interact, stop
4. Analyze timeline

// React Profiler
<React.Profiler id="Component" onRender={onRenderCallback}>
  <Component />
</React.Profiler>
```

### Node.js Profiling
```bash
# Profile CPU usage
node --prof app.js

# Analyze flame graph
node --prof-process isolate-*.log > profile.txt

# Async hook profiler
node --prof-harmony app.js
```

## Tooling

### Lighthouse
- Web performance auditing
- Accessibility
- Best practices
- SEO

### WebPageTest
- Real-world testing
- Different locations
- Network simulation
- Deep technical analysis

### Bundle Analyzer
```javascript
// Analyze bundle size
npm install --save-dev @next/bundle-analyzer
// Next.js uses webpack-bundle-analyzer
```

## Benchmarking

```javascript
// Simple benchmark
console.time('Operation');
doExpensiveOperation();
console.timeEnd('Operation');

// Benchmark.js
const Benchmark = require('benchmark');
const suite = new Benchmark.Suite;

suite
  .add('Function A', () => { /* ... */ })
  .add('Function B', () => { /* ... */ })
  .on('cycle', (event) => {
    console.log(String(event.target));
  })
  .run({ async: true });
```

## Performance Budget

### Set Performance Targets
- **First Contentful Paint**: < 1.8s
- **Largest Contentful Paint**: < 2.5s
- **Time to Interactive**: < 3.8s
- **Cumulative Layout Shift**: < 0.1
- **Total Bundle Size**: < 200KB (gzipped)

## Deliverables

- Performance optimization report
- Optimized code
- Profile analysis
- Before/after comparison
- Documentation

## Quality Checklist

- Performance is measured
- Bottlenecks are identified
- Optimizations are targeted
- No side effects introduced
- Baseline established
- Performance is monitored
- Metrics tracked over time
