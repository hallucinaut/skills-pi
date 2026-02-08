---
name: authentication
description: "Implement authentication and authorization systems. Use when adding login, registration, sessions, JWT tokens, OAuth, or security auth features."
---

# Authentication Skill

Implement authentication and authorization systems.

## When to Use

Use this skill when the user wants to:
- Implement user authentication
- Create login/register flows
- Manage sessions
- Implement JWT tokens
- Add OAuth/Social login
- Handle password reset
- Implement role-based access control

## Authentication Methods

### Session-based
- **Cookies**: Server-side sessions
- **Tokens**: Client-side sessions
- **JWT (JSON Web Token)**: Stateless authentication

### OAuth/OIDC
- **Google OAuth**: Social login
- **GitHub OAuth**: Developer tools
- **OAuth 2.0**: Generic authorization
- **OpenID Connect**: Identity layer on top of OAuth

## Common Patterns

### Password Hashing
```javascript
// Hash password
const hashedPassword = await bcrypt.hash(password, 10);

// Verify password
const isValid = await bcrypt.compare(inputPassword, hashedPassword);
```

### JWT Implementation
```javascript
// Generate token
const token = jwt.sign(
  { userId: user.id, role: user.role },
  process.env.JWT_SECRET,
  { expiresIn: '1d' }
);

// Verify token
const decoded = jwt.verify(token, process.env.JWT_SECRET);
```

### Session Management
```javascript
// Set session
req.session.userId = user.id;
req.session.userRole = user.role;

// Check session
if (!req.session.userId) {
  return res.redirect('/login');
}
```

## Security Best Practices

### Password Security
- **Never store plain text passwords**
- **Use strong hashing**: bcrypt, argon2, scrypt
- **Salting**: Unique salt per password
- **Complexity requirements**: Minimum length, character types

### JWT Security
- **Short expiration**: 15-60 minutes
- **Refresh tokens**: Longer-lived for refresh
- **HttpOnly cookies**: Prevent XSS
- **Secure flag**: HTTPS only
- **SameSite**: CSRF protection

### Session Security
- **Secure cookies**: HTTPS only
- **HttpOnly**: Prevent client-side access
- **SameSite**: CSRF protection
- **Secure identifiers**: Random, unpredictable IDs

### Rate Limiting
- **Limit login attempts**: Prevent brute force
- **Rate limit endpoints**: Prevent abuse
- **Implement on auth endpoints**

## OAuth Flows

### Authorization Code Flow
1. Redirect user to OAuth provider
2. User authenticates
3. Provider redirects back with code
4. Exchange code for access token
5. Get user profile
6. Create local user session

### Implicit Flow (deprecated)
### PKCE Flow (recommended for mobile/public)

## Email Authentication

### Password Reset
```javascript
// Generate reset token
const resetToken = crypto.randomBytes(32).toString('hex');

// Send email
await sendEmail({
  to: user.email,
  subject: 'Password Reset',
  html: `Click <a href="${resetUrl}">here</a> to reset your password`
});
```

### Email Verification
```javascript
// Send verification email
await sendEmail({
  to: user.email,
  subject: 'Verify your email',
  html: `Click <a href="${verifyUrl}">here</a> to verify`
});
```

## Authorization

### Role-Based Access Control (RBAC)
```javascript
// Check role
if (user.role !== 'admin') {
  return res.status(403).json({ error: 'Access denied' });
}
```

### Permission-Based
```javascript
// Check permission
if (!hasPermission(user, 'can_delete_users')) {
  return res.status(403).json({ error: 'Permission denied' });
}
```

### Resource Ownership
```javascript
// Check ownership
if (user.id !== resourceId) {
  return res.status(403).json({ error: 'Not your resource' });
}
```

## Common Libraries

### JavaScript
- **bcryptjs**: Password hashing
- **jsonwebtoken**: JWT handling
- **passport.js**: Authentication middleware
- **argon2**: Stronger hashing
- **jsonwebtoken-async**: Async JWT

### Python
- **Flask-Login**: Session management
- **Flask-JWT-Extended**: JWT for Flask
- **passlib**: Password hashing

### Java
- **Spring Security**: Authentication/authorization
- **BCrypt**: Password hashing

## Deliverables

- Authentication implementation
- Security configuration
- Email templates
- Error handling
- Documentation
- Integration tests

## Quality Checklist

- Passwords are hashed
- Sensitive data is logged (avoid)
- Rate limiting is implemented
- CSRF protection is configured
- HTTPS is enforced
- Session/cookie security is proper
- Error messages don't leak sensitive info
- Tests cover auth flows
