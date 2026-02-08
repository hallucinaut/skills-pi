---
name: debugging
description: "Diagnose and fix bugs. Use when tracking down errors, analyzing stack traces, reviewing logs, or implementing debugging tools."
---

# Debugging Skill

Diagnose and fix software issues efficiently.

## When to Use

Use this skill when the user wants to:
- Analyze error messages and stack traces
- Debug failing tests
- Investigate crashes and exceptions
- Review application logs
- Set up debugging tools
- Improve error handling
- Create diagnostic utilities

## Debugging Process

### 1. Reproduce the Issue
- Understand the problem statement
- Identify conditions that trigger the bug
- Create minimal reproduction case

### 2. Gather Information
- Review logs and error messages
- Check stack traces
- Inspect application state
- Review configuration

### 3. Analyze the Code
- Trace execution path
- Identify logical errors
- Check for edge cases
- Verify assumptions

### 4. Hypothesize Causes
- List potential root causes
- Prioritize likely issues
- Form testable hypotheses

### 5. Test Hypotheses
- Add debug logging
- Use breakpoints
- Modify behavior intentionally
- Verify predictions

### 6. Fix and Verify
- Implement fix
- Test fix thoroughly
- Verify no regressions
- Document the fix

## Debugging Techniques

### For Code
- **Breakpoints**: Pause execution at specific points
- **Stepping**: Move through code line by line
- **Watch variables**: Monitor variable values
- **Call stacks**: Understand call hierarchy
- **Linter tools**: Catch syntax errors

### For Logs
- **Structured logging**: Organized log entries
- **Correlation IDs**: Track requests through system
- **Log levels**: Filter by importance
- **Log rotation**: Manage log file size
- **Centralized logging**: Aggregated log storage

### For Production
- **Error tracking**: Production error monitoring
- **Performance profiling**: Identify bottlenecks
- **Resource usage**: Monitor memory, CPU, network
- **Distributed tracing**: Track requests across services

## Common Bug Patterns

### Logic Errors
- Off-by-one errors
- Incorrect conditionals
- Wrong operators
- Edge case failures

### Resource Issues
- Memory leaks
- Infinite loops
- Deadlocks
- Resource exhaustion

### Integration Errors
- API mismatches
- Protocol incompatibilities
- Wrong data types
- Timing issues

### Configuration Errors
- Wrong values
- Missing environment variables
- Misconfigured settings
- Dependency issues

## Best Practices

- **Log everything important**: Errors, warnings, relevant state changes
- **Make logs searchable**: Structured format, meaningful keys
- **Use meaningful error messages**: Explain what and why
- **Add context**: Request ID, user info, timestamps
- **Don't log secrets**: Never log passwords, tokens, API keys
- **Add assertions**: Fail fast when assumptions are violated

## Debugging Tools

### IDE Tools
- Breakpoints
- Debugger
- Linting
- IDE features

### Runtime Tools
- Profilers
- Memory profilers
- Tracing tools
- Network inspection

### Production Tools
- Error tracking
- Monitoring
- Logging aggregation
- APM (Application Performance Monitoring)

## Deliverables

- Bug diagnosis report
- Fix implementation
- Test cases for the bug
- Updated documentation
- Debugging utilities (if needed)

## Quality Checklist

- Bug is reproducible with clear steps
- Root cause is identified
- Fix addresses the root cause
- No regressions introduced
- Tests are added for the fix
- Documentation is updated
