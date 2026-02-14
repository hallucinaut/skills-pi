---
name: api-integration
description: "Integrate with third-party APIs, webhooks, OAuth flows, and external services. Handle authentication, rate limiting, retries, and error handling."
---

# API Integration Skill

Build robust integrations with third-party APIs and external services.

## When to Use

Use this skill when the user wants to:
- Integrate with third-party APIs (REST, GraphQL, SOAP)
- Implement OAuth 2.0 flows
- Handle webhook integrations
- Build API clients and SDKs
- Implement rate limiting and backoff strategies
- Handle API authentication (API keys, JWT, OAuth)
- Work with payment gateways (Stripe, PayPal)
- Integrate with SaaS platforms (Salesforce, HubSpot, Slack)
- Process webhooks securely
- Implement API retry logic and circuit breakers

## Authentication Methods

### 1. API Keys
```python
import requests

headers = {
    'Authorization': f'Bearer {API_KEY}',
    'Content-Type': 'application/json'
}

response = requests.get('https://api.example.com/data', headers=headers)
```

### 2. OAuth 2.0
```python
from requests_oauthlib import OAuth2Session

# Authorization flow
oauth = OAuth2Session(client_id, redirect_uri=redirect_uri, scope=scope)
authorization_url, state = oauth.authorization_url(
    'https://provider.com/oauth/authorize'
)

# Token exchange
token = oauth.fetch_token(
    'https://provider.com/oauth/token',
    client_secret=client_secret,
    authorization_response=callback_url
)

# API calls with token
response = oauth.get('https://api.provider.com/user')
```

### 3. JWT (JSON Web Tokens)
```python
import jwt
import time

def create_jwt_token(api_key, secret_key):
    payload = {
        'iss': api_key,
        'iat': int(time.time()),
        'exp': int(time.time()) + 3600
    }
    return jwt.encode(payload, secret_key, algorithm='HS256')

headers = {
    'Authorization': f'Bearer {create_jwt_token(API_KEY, SECRET)}'
}
```

### 4. Basic Auth
```python
from requests.auth import HTTPBasicAuth

response = requests.get(
    'https://api.example.com/data',
    auth=HTTPBasicAuth('username', 'password')
)
```

## API Client Best Practices

### Robust Client Implementation
```python
import requests
from typing import Optional, Dict, Any
import time
from functools import wraps

class APIClient:
    def __init__(self, base_url: str, api_key: str, timeout: int = 30):
        self.base_url = base_url.rstrip('/')
        self.api_key = api_key
        self.timeout = timeout
        self.session = requests.Session()
        self.session.headers.update({
            'Authorization': f'Bearer {api_key}',
            'User-Agent': 'MyApp/1.0',
            'Accept': 'application/json'
        })

    def _retry_with_backoff(self, max_retries=3):
        """Decorator for retry logic with exponential backoff."""
        def decorator(func):
            @wraps(func)
            def wrapper(*args, **kwargs):
                for attempt in range(max_retries):
                    try:
                        return func(*args, **kwargs)
                    except requests.exceptions.RequestException as e:
                        if attempt == max_retries - 1:
                            raise
                        wait_time = 2 ** attempt
                        time.sleep(wait_time)
            return wrapper
        return decorator

    @_retry_with_backoff(max_retries=3)
    def _request(self, method: str, endpoint: str, **kwargs) -> Dict[Any, Any]:
        """Make HTTP request with error handling."""
        url = f"{self.base_url}/{endpoint.lstrip('/')}"

        try:
            response = self.session.request(
                method,
                url,
                timeout=self.timeout,
                **kwargs
            )
            response.raise_for_status()
            return response.json()

        except requests.exceptions.HTTPError as e:
            if response.status_code == 429:  # Rate limit
                retry_after = int(response.headers.get('Retry-After', 60))
                raise RateLimitError(f"Rate limited. Retry after {retry_after}s")
            elif response.status_code == 401:
                raise AuthenticationError("Invalid API credentials")
            elif response.status_code == 404:
                raise NotFoundError(f"Resource not found: {url}")
            else:
                raise APIError(f"API error: {e}")

        except requests.exceptions.Timeout:
            raise APIError("Request timeout")

        except requests.exceptions.ConnectionError:
            raise APIError("Connection error")

    def get(self, endpoint: str, params: Optional[Dict] = None) -> Dict:
        """GET request."""
        return self._request('GET', endpoint, params=params)

    def post(self, endpoint: str, data: Optional[Dict] = None) -> Dict:
        """POST request."""
        return self._request('POST', endpoint, json=data)

    def put(self, endpoint: str, data: Optional[Dict] = None) -> Dict:
        """PUT request."""
        return self._request('PUT', endpoint, json=data)

    def delete(self, endpoint: str) -> Dict:
        """DELETE request."""
        return self._request('DELETE', endpoint)

# Custom exceptions
class APIError(Exception):
    pass

class RateLimitError(APIError):
    pass

class AuthenticationError(APIError):
    pass

class NotFoundError(APIError):
    pass
```

### Rate Limiting Handler
```python
from datetime import datetime, timedelta
import threading

class RateLimiter:
    def __init__(self, max_requests: int, time_window: int):
        self.max_requests = max_requests
        self.time_window = time_window  # seconds
        self.requests = []
        self.lock = threading.Lock()

    def allow_request(self) -> bool:
        """Check if request is allowed under rate limit."""
        with self.lock:
            now = datetime.now()
            cutoff = now - timedelta(seconds=self.time_window)

            # Remove old requests
            self.requests = [req for req in self.requests if req > cutoff]

            if len(self.requests) < self.max_requests:
                self.requests.append(now)
                return True
            return False

    def wait_if_needed(self):
        """Wait until rate limit allows request."""
        while not self.allow_request():
            time.sleep(0.1)

# Usage
rate_limiter = RateLimiter(max_requests=100, time_window=60)

def api_call():
    rate_limiter.wait_if_needed()
    # Make API request
    return client.get('/endpoint')
```

## Webhook Integration

### Receiving Webhooks
```python
from flask import Flask, request, jsonify
import hmac
import hashlib

app = Flask(__name__)
WEBHOOK_SECRET = 'your-webhook-secret'

def verify_webhook_signature(payload: bytes, signature: str) -> bool:
    """Verify webhook signature for security."""
    expected_signature = hmac.new(
        WEBHOOK_SECRET.encode(),
        payload,
        hashlib.sha256
    ).hexdigest()
    return hmac.compare_digest(f'sha256={expected_signature}', signature)

@app.route('/webhooks/stripe', methods=['POST'])
def stripe_webhook():
    """Handle Stripe webhook."""
    payload = request.get_data()
    signature = request.headers.get('Stripe-Signature')

    # Verify signature
    if not verify_webhook_signature(payload, signature):
        return jsonify({'error': 'Invalid signature'}), 401

    # Process webhook
    event = request.json
    event_type = event.get('type')

    if event_type == 'payment_intent.succeeded':
        handle_payment_success(event['data']['object'])
    elif event_type == 'payment_intent.failed':
        handle_payment_failure(event['data']['object'])

    return jsonify({'received': True}), 200

def handle_payment_success(payment_intent):
    """Process successful payment."""
    # Update database, send confirmation email, etc.
    pass
```

### Webhook Retry Logic
```python
import requests
from celery import Celery

app = Celery('webhooks', broker='redis://localhost:6379')

@app.task(bind=True, max_retries=5)
def send_webhook(self, url: str, payload: dict, signature: str):
    """Send webhook with retry logic."""
    try:
        response = requests.post(
            url,
            json=payload,
            headers={'X-Webhook-Signature': signature},
            timeout=10
        )
        response.raise_for_status()
        return {'status': 'success', 'status_code': response.status_code}

    except requests.exceptions.RequestException as exc:
        # Exponential backoff: 1m, 5m, 25m, 2h, 10h
        countdown = 60 * (5 ** self.request.retries)
        raise self.retry(exc=exc, countdown=countdown)
```

## Common Integrations

### Stripe Payment Integration
```python
import stripe

stripe.api_key = 'sk_test_...'

def create_payment_intent(amount: int, currency: str = 'usd'):
    """Create a payment intent."""
    try:
        intent = stripe.PaymentIntent.create(
            amount=amount,
            currency=currency,
            metadata={'order_id': '12345'}
        )
        return intent

    except stripe.error.CardError as e:
        # Card declined
        raise PaymentError(f"Card declined: {e.user_message}")

    except stripe.error.RateLimitError:
        raise PaymentError("Too many requests to Stripe")

    except stripe.error.InvalidRequestError as e:
        raise PaymentError(f"Invalid parameters: {e}")

    except stripe.error.AuthenticationError:
        raise PaymentError("Authentication failed")

    except stripe.error.StripeError as e:
        raise PaymentError(f"Stripe error: {e}")
```

### Slack Integration
```python
from slack_sdk import WebClient
from slack_sdk.errors import SlackApiError

client = WebClient(token='xoxb-...')

def send_slack_message(channel: str, message: str):
    """Send message to Slack channel."""
    try:
        response = client.chat_postMessage(
            channel=channel,
            text=message,
            blocks=[
                {
                    "type": "section",
                    "text": {"type": "mrkdwn", "text": message}
                }
            ]
        )
        return response

    except SlackApiError as e:
        print(f"Error: {e.response['error']}")
        raise
```

### SendGrid Email Integration
```python
from sendgrid import SendGridAPIClient
from sendgrid.helpers.mail import Mail

def send_email(to_email: str, subject: str, content: str):
    """Send email via SendGrid."""
    message = Mail(
        from_email='noreply@example.com',
        to_emails=to_email,
        subject=subject,
        html_content=content
    )

    try:
        sg = SendGridAPIClient(api_key='SG...')
        response = sg.send(message)
        return response.status_code

    except Exception as e:
        print(f"Error sending email: {e}")
        raise
```

### Google APIs Integration
```python
from google.oauth2.credentials import Credentials
from googleapiclient.discovery import build

def get_google_calendar_events(credentials: Credentials):
    """Fetch calendar events from Google Calendar API."""
    try:
        service = build('calendar', 'v3', credentials=credentials)

        events_result = service.events().list(
            calendarId='primary',
            maxResults=10,
            singleEvents=True,
            orderBy='startTime'
        ).execute()

        return events_result.get('items', [])

    except Exception as e:
        print(f"Error fetching calendar events: {e}")
        raise
```

## Pagination Handling

### Cursor-Based Pagination
```python
def fetch_all_pages(endpoint: str, limit: int = 100):
    """Fetch all pages using cursor pagination."""
    all_results = []
    cursor = None

    while True:
        params = {'limit': limit}
        if cursor:
            params['cursor'] = cursor

        response = client.get(endpoint, params=params)
        all_results.extend(response['data'])

        cursor = response.get('next_cursor')
        if not cursor:
            break

    return all_results
```

### Offset-Based Pagination
```python
def fetch_all_pages_offset(endpoint: str, page_size: int = 100):
    """Fetch all pages using offset pagination."""
    all_results = []
    offset = 0

    while True:
        params = {'limit': page_size, 'offset': offset}
        response = client.get(endpoint, params=params)

        results = response['data']
        all_results.extend(results)

        if len(results) < page_size:
            break

        offset += page_size

    return all_results
```

## Error Handling & Retries

### Circuit Breaker Pattern
```python
from datetime import datetime, timedelta

class CircuitBreaker:
    def __init__(self, failure_threshold=5, recovery_timeout=60):
        self.failure_threshold = failure_threshold
        self.recovery_timeout = recovery_timeout
        self.failures = 0
        self.last_failure_time = None
        self.state = 'CLOSED'  # CLOSED, OPEN, HALF_OPEN

    def call(self, func, *args, **kwargs):
        if self.state == 'OPEN':
            if datetime.now() - self.last_failure_time > timedelta(seconds=self.recovery_timeout):
                self.state = 'HALF_OPEN'
            else:
                raise Exception("Circuit breaker is OPEN")

        try:
            result = func(*args, **kwargs)
            self.on_success()
            return result
        except Exception as e:
            self.on_failure()
            raise

    def on_success(self):
        self.failures = 0
        self.state = 'CLOSED'

    def on_failure(self):
        self.failures += 1
        self.last_failure_time = datetime.now()
        if self.failures >= self.failure_threshold:
            self.state = 'OPEN'
```

## Testing API Integrations

### Mocking External APIs
```python
import pytest
from unittest.mock import Mock, patch
import requests_mock

def test_api_client():
    """Test API client with mocked responses."""
    with requests_mock.Mocker() as m:
        m.get('https://api.example.com/users', json={'users': []})

        client = APIClient('https://api.example.com', 'test-key')
        result = client.get('/users')

        assert result == {'users': []}

def test_rate_limit_handling():
    """Test rate limit error handling."""
    with requests_mock.Mocker() as m:
        m.get(
            'https://api.example.com/data',
            status_code=429,
            headers={'Retry-After': '60'}
        )

        client = APIClient('https://api.example.com', 'test-key')

        with pytest.raises(RateLimitError):
            client.get('/data')
```

## Best Practices

### Security
- **Never expose API keys** in code or logs
- **Use environment variables** for credentials
- **Verify webhook signatures** to prevent spoofing
- **Use HTTPS only** for API calls
- **Implement request signing** for critical operations
- **Rotate API keys regularly**

### Reliability
- **Implement retries** with exponential backoff
- **Use circuit breakers** to prevent cascading failures
- **Handle rate limits** gracefully
- **Log all API interactions** for debugging
- **Monitor API health** and latency
- **Cache responses** when appropriate

### Performance
- **Use connection pooling**
- **Implement pagination** for large datasets
- **Batch requests** when possible
- **Use async requests** for parallel operations
- **Compress request/response** payloads

### Maintainability
- **Version your API clients**
- **Document API endpoints** and parameters
- **Handle API versioning** properly
- **Create typed models** for responses
- **Write integration tests**

## Deliverables

- API client implementation
- Authentication handling
- Retry and error handling logic
- Webhook receivers (if applicable)
- Rate limiting implementation
- Test suite with mocked responses
- Documentation and examples
- Monitoring and logging

## Quality Checklist

- Authentication is secure and configurable
- Error handling covers all failure modes
- Retry logic with exponential backoff
- Rate limiting is implemented
- Webhook signatures are verified
- All API calls are logged
- Integration tests are written
- API credentials are in environment variables
- Circuit breaker protects against failures
- Documentation includes all endpoints
