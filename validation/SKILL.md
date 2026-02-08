---
name: validation
description: "Validate input data, ensure data integrity, and implement data checking rules. Use when validating forms, API inputs, data models, or user input."
---

# Validation Skill

Validate input data and ensure data integrity.

## When to Use

Use this skill when the user wants to:
- Validate user inputs
- Check data types
- Enforce data format rules
- Implement validation schemas
- Validate API request/response
- Ensure data consistency
- Provide user-friendly validation errors

## Validation Types

### Client-Side Validation
- **Form validation**: Before submission
- **Input type validation**: Email, URL, date
- **Format validation**: Email, phone, zip code
- **Length validation**: Min/max length
- **Pattern validation**: Regex patterns

### Server-Side Validation
- **API request validation**: All requests
- **Data model validation**: Database constraints
- **Business rules validation**: Custom rules
- **Security validation**: No injection attacks

### Data Integrity Validation
- **Format validation**: JSON, XML, CSV
- **Type validation**: Correct types
- **Range validation**: Value ranges
- **Required fields**: All required fields present

## Common Validation Rules

### Email Validation
```javascript
// Email pattern
const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;

// Validate
if (!emailRegex.test(email)) {
  return { error: 'Invalid email format' };
}
```

### Password Validation
```javascript
// Password requirements
const password = req.body.password;
const isValid =
  password.length >= 8 &&
  /[A-Z]/.test(password) &&
  /[0-9]/.test(password) &&
  /[^A-Za-z0-9]/.test(password);
```

### URL Validation
```javascript
// URL regex
const urlRegex = /^https?:\/\/.+/;

if (!urlRegex.test(url)) {
  return { error: 'Invalid URL format' };
}
```

### Phone Validation
```javascript
// Phone regex (US format)
const phoneRegex = /^\d{3}-\d{3}-\d{4}$/;

if (!phoneRegex.test(phone)) {
  return { error: 'Invalid phone format (use XXX-XXX-XXXX)' };
}
```

## Validation Libraries

### JavaScript
- **Joi**: Schema validation
- **Zod**: Runtime type validation
- **Yup**: Schema validation
- **validator.js**: Utility functions
- **ajv**: JSON Schema validation

### Python
- **Pydantic**: Data validation
- **Marshmallow**: Data validation and serialization
- **email-validator**: Email validation
- **validators**: Utility validators

## Schema Validation

### Joi Schema
```javascript
const Joi = require('joi');

const schema = Joi.object({
  email: Joi.string()
    .email()
    .required(),
  age: Joi.number()
    .integer()
    .min(18)
    .max(120)
    .required(),
  username: Joi.string()
    .alphanum()
    .min(3)
    .max(30)
    .required(),
  password: Joi.string()
    .min(8)
    .pattern(new RegExp('(?=.*[A-Z])'))
    .pattern(new RegExp('(?=.*[0-9])')),
});
```

### Pydantic Model
```python
from pydantic import BaseModel, EmailStr, validator

class User(BaseModel):
    email: EmailStr
    age: int
    username: str
    password: str

    @validator('age')
    def validate_age(cls, v):
        if v < 18 or v > 120:
            raise ValueError('Age must be between 18 and 120')
        return v
```

## API Validation

### Request Validation
```javascript
// Express middleware
const validate = (schema) => (req, res, next) => {
  const { error, value } = schema.validate(req.body);
  if (error) {
    return res.status(400).json({
      error: 'Validation failed',
      details: error.details
    });
  }
  req.body = value;
  next();
};

// Usage
app.post('/users', validate(userSchema), createUser);
```

### Response Validation
```javascript
// Validate API responses
const validateResponse = (schema) => (data) => {
  const { error } = schema.validate(data);
  if (error) {
    console.error('Invalid response:', error.details);
    throw new Error('Response validation failed');
  }
  return data;
};
```

## Custom Validators

### Conditional Validation
```javascript
// If field is present, validate it
if (req.body.nickname) {
  if (req.body.nickname.length > 50) {
    return { error: 'Nickname too long' };
  }
}
```

### Cross-Field Validation
```javascript
// Password must match confirmation
if (req.body.password !== req.body.confirmPassword) {
  return { error: 'Passwords do not match' };
}
```

### Complex Rules
```javascript
// Date range validation
const startDate = new Date(req.body.startDate);
const endDate = new Date(req.body.endDate);

if (startDate >= endDate) {
  return { error: 'End date must be after start date' };
}
```

## Error Messages

### User-Friendly Errors
```javascript
{
  email: [
    { type: 'string.empty', message: 'Email is required' },
    { type: 'string.email', message: 'Invalid email format' }
  ],
  age: 'Age must be between 18 and 120'
}
```

### Technical Errors (Hidden)
```javascript
// Don't expose validation internals
{
  error: 'Validation failed',
  field: 'email',
  message: 'Invalid email format'
}
```

## Validation Best Practices

- **Fail fast**: Validate early in the request
- **Be specific**: Clear, actionable error messages
- **Server-side**: Never trust client validation
- **Consistent**: Same rules everywhere
- **Document**: Validate input as documented
- **Type hints**: Use type annotations

## Deliverables

- Validation schemas
- Error handling implementation
- Middleware components
- Documentation
- Test cases

## Quality Checklist

- Validation rules are enforced
- Error messages are clear
- Client validation doesn't replace server validation
- Invalid data is handled gracefully
- Sanitization is applied
- Validators are reusable
- Edge cases are tested
