---
name: security-audit
description: "Perform security assessments, vulnerability scanning, and penetration testing for codebases, APIs, and infrastructure. Use when conducting security reviews, penetration tests, or compliance assessments."
---

# Security Audit Skill

Comprehensive security assessment and vulnerability detection for applications and systems.

## When to Use

Use this skill when the user wants to:
- Conduct security code reviews
- Perform penetration testing
- Scan for known vulnerabilities
- Evaluate security posture
- Check compliance requirements
- Identify data protection issues

## Audit Scope

### Application Security
- Input validation and sanitization
- Authentication and authorization
- Session management
- Cryptographic practices
- File upload/download security
- API security (authentication, rate limiting, input validation)

### Infrastructure Security
- Network configurations
- Container/VM hardening
- Configuration management
- Dependency vulnerabilities
- Secrets management

### Data Security
- Data at rest encryption
- Data in transit encryption
- PII/PHI handling
- Data breach response

### Security Controls
- Logging and monitoring
- Error handling (don't expose internals)
- Security headers
- Content Security Policy
- XSS protection

## Deliverables

- Executive summary with risk rating
- Detailed vulnerability findings
- Remediation recommendations
- Priority-ordered action items
- Compliance checklist (if applicable)
- Proof-of-concept exploit code (if authorized)

## Reporting Format

- **Severity levels**: Critical, High, Medium, Low
- **CVSS scores** where applicable
- **Remediation guidance** with code examples
- **Reference URLs** for vulnerability details
- **Evidence** (screenshot, log, code snippet)

## Quality Checklist

- All high and critical vulnerabilities addressed
- Remediation steps are clear and actionable
- Code examples are complete and executable
- Risk rating is justified
- Compliance requirements are met
- Report is well-structured and readable
