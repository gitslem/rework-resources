# API Integration Blueprint: REST, OAuth & Webhooks

## A Reusable Architecture for Building Secure, Reliable API Integrations

---

## Architecture Overview

```
Client Application
    ↓
Request Handler (rate limiting, retries)
    ↓
Authentication (OAuth 2.0, API Key)
    ↓
API Gateway (transformation, logging)
    ↓
Third-Party API
    ↓
Response Handler (parsing, validation)
    ↓
Error Handler (retries, fallbacks)
    ↓
Cache Layer (Redis, in-memory)
    ↓
Application Database
```

---

## 1. Authentication Patterns

### OAuth 2.0 Authorization Code Flow

```
1. User clicks "Login with Provider"
2. Client redirects to: https://provider.com/oauth/authorize?
   client_id=YOUR_CLIENT_ID&
   redirect_uri=YOUR_REDIRECT_URI&
   scope=user:email,repo&
   state=RANDOM_STATE_STRING

3. User approves, provider redirects to: YOUR_REDIRECT_URI?code=AUTH_CODE&state=STATE

4. Exchange code for token:
   POST /oauth/token
   {
     "grant_type": "authorization_code",
     "code": "AUTH_CODE",
     "client_id": "YOUR_CLIENT_ID",
     "client_secret": "YOUR_SECRET",
     "redirect_uri": "YOUR_REDIRECT_URI"
   }

5. Response:
   {
     "access_token": "TOKEN_STRING",
     "token_type": "Bearer",
     "expires_in": 3600,
     "refresh_token": "REFRESH_TOKEN"
   }

6. Use token in requests:
   Authorization: Bearer ACCESS_TOKEN
```

**Best Practice:** Store tokens encrypted, never in code

### API Key Authentication

```
// Most insecure: avoid
GET /api/users?api_key=ABC123

// Better: use header
GET /api/users
Authorization: Bearer ABC123

// Best: rotate keys, use short-lived tokens
GET /api/users
X-API-Key: production_key_20260410
```

**Best Practice:** Rotate keys every 90 days, use separate keys per environment

### Mutual TLS (mTLS)

```
// For high-security integrations
1. Generate client certificate
2. Configure certificate on both client and server
3. Server validates client certificate
4. Client validates server certificate

Benefit: Certificate pinning, harder to intercept
Use when: Handling PII, financial data, healthcare info
```

---

## 2. Request & Response Handling

### Standardized Request Pattern

```javascript
const apiRequest = async (method, endpoint, data, options = {}) => {
  const { retries = 3, timeout = 5000, cache = false } = options;

  // Check cache first
  if (cache && method === 'GET') {
    const cached = getFromCache(endpoint);
    if (cached && !isExpired(cached)) return cached.data;
  }

  // Build request
  const config = {
    method,
    url: `${API_BASE_URL}${endpoint}`,
    headers: {
      'Authorization': `Bearer ${getToken()}`,
      'Content-Type': 'application/json',
      'User-Agent': 'MyApp/1.0',
      'X-Request-ID': generateUUID() // for tracking
    },
    data,
    timeout
  };

  // Execute with retries
  for (let attempt = 1; attempt <= retries; attempt++) {
    try {
      const response = await axios(config);
      
      // Cache successful response
      if (cache && method === 'GET') {
        setInCache(endpoint, response.data, 300000); // 5 min TTL
      }
      
      return response.data;
    } catch (error) {
      if (attempt === retries) throw error;
      
      // Exponential backoff
      const delay = Math.pow(2, attempt - 1) * 1000;
      await sleep(delay);
    }
  }
};
```

### Response Validation Pattern

```javascript
const validateResponse = (response, schema) => {
  // Check for required fields
  for (const field of schema.required) {
    if (!(field in response)) {
      throw new Error(`Missing required field: ${field}`);
    }
  }

  // Type checking
  for (const [field, type] of Object.entries(schema.fields)) {
    if (typeof response[field] !== type) {
      throw new Error(`Invalid type for ${field}: expected ${type}`);
    }
  }

  // Range validation
  if (schema.ranges) {
    for (const [field, range] of Object.entries(schema.ranges)) {
      if (response[field] < range.min || response[field] > range.max) {
        throw new Error(`${field} out of range`);
      }
    }
  }

  return true;
};

// Usage
const userSchema = {
  required: ['id', 'email', 'name'],
  fields: {
    id: 'number',
    email: 'string',
    name: 'string',
    active: 'boolean'
  }
};
```

---

## 3. Error Handling & Retry Logic

### Retry Strategy

```javascript
const shouldRetry = (error) => {
  const { status, code } = error;
  
  // Retry on network errors
  if (code === 'ECONNREFUSED' || code === 'ENOTFOUND') return true;
  
  // Retry on server errors (5xx)
  if (status >= 500) return true;
  
  // Retry on rate limiting (429)
  if (status === 429) return true;
  
  // Retry on timeout
  if (code === 'ECONNABORTED') return true;
  
  // Don't retry on client errors (4xx, except 429)
  if (status >= 400 && status < 500) return false;
  
  return false;
};

// Exponential backoff with jitter
const getBackoffDelay = (attempt, baseDelay = 1000) => {
  const exponential = baseDelay * Math.pow(2, attempt - 1);
  const jitter = Math.random() * 0.1 * exponential; // 10% jitter
  return exponential + jitter;
};
```

### Error Response Handler

```javascript
const handleError = (error, context) => {
  const errorResponse = {
    timestamp: new Date().toISOString(),
    endpoint: context.endpoint,
    status: error.response?.status || 500,
    message: error.message,
    requestId: context.requestId
  };

  // Log for monitoring
  logger.error(errorResponse);

  // Send to error tracking (Sentry, DataDog, etc)
  if (error.response?.status >= 500) {
    sendToErrorTracking(errorResponse);
  }

  // Fallback behavior
  if (context.fallback) {
    return context.fallback;
  }

  // Retry if applicable
  if (shouldRetry(error) && context.retryCount < context.maxRetries) {
    return retry(context);
  }

  throw error;
};
```

---

## 4. Webhook Handling

### Webhook Receiver Pattern

```javascript
// Express route for receiving webhooks
app.post('/webhooks/provider', verifyWebhookSignature, async (req, res) => {
  const { id, type, data, timestamp } = req.body;

  // Prevent replay attacks: check timestamp
  if (Date.now() - new Date(timestamp).getTime() > 5 * 60 * 1000) {
    return res.status(400).json({ error: 'Webhook too old' });
  }

  // Check for duplicate
  if (await isProcessed(id)) {
    return res.status(200).json({ status: 'already processed' });
  }

  try {
    // Queue for async processing (don't process in request)
    await queue.add('process_webhook', { type, data, id });
    
    // Respond immediately (webhook expects 2xx within 5 seconds)
    res.status(200).json({ received: true });
  } catch (error) {
    logger.error({ error, webhook: id });
    res.status(500).json({ error: 'Processing failed' });
  }
});

// Verify webhook signature (HMAC-SHA256 is common)
const verifyWebhookSignature = (req, res, next) => {
  const signature = req.headers['x-webhook-signature'];
  const timestamp = req.headers['x-webhook-timestamp'];
  
  const payload = JSON.stringify(req.body);
  const hmac = crypto
    .createHmac('sha256', WEBHOOK_SECRET)
    .update(`${timestamp}.${payload}`)
    .digest('hex');

  if (hmac !== signature) {
    return res.status(401).json({ error: 'Invalid signature' });
  }

  next();
};
```

### Webhook Consumer Queue

```javascript
// Bull or similar queue processor
queue.process('process_webhook', async (job) => {
  const { type, data, id } = job.data;

  switch (type) {
    case 'user.created':
      await handleUserCreated(data);
      break;
    case 'payment.completed':
      await handlePaymentCompleted(data);
      break;
    default:
      logger.warn(`Unknown webhook type: ${type}`);
  }

  // Mark as processed
  await markProcessed(id);
});

// Retry on failure
queue.on('failed', async (job, err) => {
  logger.error({ job: job.id, error: err.message });
  
  // Exponential backoff: retry at 1min, 5min, 30min
  if (job.attemptsMade < 3) {
    await job.retry();
  } else {
    // Send alert if all retries exhausted
    await alertOps(`Webhook ${job.id} failed after 3 retries`);
  }
});
```

---

## 5. Rate Limiting

### Client-Side Rate Limiting

```javascript
const rateLimiter = new Map();

const isRateLimited = (endpoint) => {
  const key = endpoint;
  const now = Date.now();
  
  if (!rateLimiter.has(key)) {
    rateLimiter.set(key, [now]);
    return false;
  }

  const times = rateLimiter.get(key);
  const recentRequests = times.filter(t => now - t < 60000); // Last 60 seconds

  if (recentRequests.length >= 60) { // 60 requests per minute
    return true;
  }

  recentRequests.push(now);
  rateLimiter.set(key, recentRequests);
  return false;
};

// Usage
if (isRateLimited(endpoint)) {
  // Wait and retry
  await sleep(1000);
  return apiRequest(...args);
}
```

### Server-Side Rate Limiting

```javascript
// Use express-rate-limit middleware
const rateLimit = require('express-rate-limit');

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // 100 requests per window
  message: 'Too many requests, please try again later',
  standardHeaders: true, // Return rate limit info in headers
  legacyHeaders: false
});

app.use('/api/', limiter);
```

---

## 6. Caching Strategy

### Cache Implementation

```javascript
const cache = new Map();

const getCached = (key, ttl = 300000) => {
  const cached = cache.get(key);
  if (!cached) return null;
  
  if (Date.now() - cached.timestamp > ttl) {
    cache.delete(key);
    return null;
  }
  
  return cached.data;
};

const setCached = (key, data) => {
  cache.set(key, {
    data,
    timestamp: Date.now()
  });
};

// Usage in API request
const getUser = async (userId) => {
  const cacheKey = `user_${userId}`;
  
  // Check cache first (5 min TTL)
  const cached = getCached(cacheKey, 5 * 60 * 1000);
  if (cached) return cached;

  // Fetch from API
  const user = await apiRequest('GET', `/users/${userId}`);
  
  // Cache result
  setCached(cacheKey, user);
  
  return user;
};
```

### Cache Invalidation

```javascript
// Invalidate on mutation
const updateUser = async (userId, updates) => {
  const response = await apiRequest('PUT', `/users/${userId}`, updates);
  
  // Clear related caches
  cache.delete(`user_${userId}`);
  cache.delete('users_list'); // Bust list cache too
  
  return response;
};
```

---

## 7. Monitoring & Logging

### Structured Logging

```javascript
const logApiRequest = (context) => {
  const logEntry = {
    timestamp: new Date().toISOString(),
    level: 'info',
    endpoint: context.endpoint,
    method: context.method,
    status: context.status,
    duration: context.duration,
    requestId: context.requestId,
    userId: context.userId,
    error: context.error
  };

  // Use structured logging (JSON)
  console.log(JSON.stringify(logEntry));
};

// Usage
const startTime = Date.now();
try {
  const response = await apiRequest('GET', '/users');
  logApiRequest({
    endpoint: '/users',
    method: 'GET',
    status: 200,
    duration: Date.now() - startTime,
    requestId: req.id
  });
} catch (error) {
  logApiRequest({
    endpoint: '/users',
    method: 'GET',
    status: error.status,
    duration: Date.now() - startTime,
    requestId: req.id,
    error: error.message
  });
}
```

### Monitoring Metrics

```
Key metrics to track:
1. Request latency (p50, p95, p99)
2. Error rate (by status code)
3. Rate limit hits
4. Cache hit ratio
5. Retry rate
6. Webhook delivery success rate
7. Queue backlog size

Alert thresholds:
- Latency p99 > 5 seconds: investigate
- Error rate > 1%: alert ops
- Queue backlog > 10,000: scale up
```

---

## Implementation Checklist

- [ ] Define authentication method (OAuth, API Key, mTLS)
- [ ] Implement request/response handlers
- [ ] Add retry logic with exponential backoff
- [ ] Implement rate limiting
- [ ] Set up webhook receiver with signature verification
- [ ] Add comprehensive error handling
- [ ] Implement caching strategy
- [ ] Set up logging and monitoring
- [ ] Test failure scenarios (network down, API down, rate limit)
- [ ] Document all endpoints and error codes
- [ ] Set up alerts for production issues

---

## About Rework Digital

This blueprint was created by **Rework Digital** - Resources Department for automation professionals.

**Resources Department Contact:** resource@reworkdigital.io  
**Follow us on GitHub:** https://github.com/Reworkdigital-io

---

*Last Updated: 2026-04-10*  
*Version: 1.0*
