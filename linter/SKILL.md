---
name: linter
description: "Apply code linting, formatting, and style checks. Use when enforcing code style, fixing formatting issues, or running static analysis."
---

# Linter Skill

Apply code linting, formatting, and style checks to improve code quality and consistency.

## When to Use

Use this skill when the user wants to:
- Enforce code style and formatting
- Fix linting errors
- Configure linters
- Run static analysis
- Fix formatting issues
- Check for code style violations

## Linter Tools

### JavaScript/TypeScript
- **ESLint**: Linting and code style
- **Prettier**: Code formatting
- **Husky**: Git hooks
- **lint-staged**: Lint staged files

### Python
- **flake8**: Linting
- **black**: Formatting
- **isort**: Import sorting
- **pylint**: Static analysis

### Java
- **Checkstyle**: Linting
- **SpotBugs**: Static analysis
- **PMD**: Static analysis

### General
- **Standard**: JavaScript style
- **RuboCop**: Ruby style
- **Go fmt**: Go formatting

## Common Linting Rules

### JavaScript/TypeScript
```javascript
// ESLint rules example
rules: {
  'no-console': 'warn',
  'no-unused-vars': 'error',
  'prefer-const': 'error',
  'quotes': ['error', 'single'],
  'semi': ['error', 'always']
}
```

### Python
```python
# flake8 configuration
[flake8]
max-line-length = 100
ignore = E203, W503
exclude = .git, __pycache__, venv
```

## Formatting Tools

### Prettier Configuration
```javascript
// .prettierrc
{
  "semi": true,
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "es5",
  "printWidth": 80
}
```

### Black Configuration
```python
# pyproject.toml
[tool.black]
line-length = 100
target-version = ['py39']
```

## Integration with Git

### Pre-commit Hooks
```bash
# Install husky for git hooks
npm install husky lint-staged --save-dev
npx husky install
npx husky add .husky/pre-commit "npx lint-staged"
```

### lint-staged
```javascript
// .lintstagedrc
{
  "*.{js,jsx,ts,tsx}": ["eslint --fix", "prettier --write"],
  "*.{json,md,css,scss}": ["prettier --write"]
}
```

## Linting Best Practices

- **Automatic**: Run on save, not just CI
- **Fail fast**: Fail build on critical issues
- **Fixable**: Auto-fix what's possible
- **Configurable**: Custom rules for project
- **Team-aligned**: Consistent style across team

## Deliverables

- Linter configuration files
- Formatters setup
- Pre-commit hooks
- CI integration
- Documentation

## Quality Checklist

- Linters are configured
- Auto-fixes are applied
- Code style is consistent
- No style violations in CI
- Configuration matches project guidelines
- Documentation explains rules
