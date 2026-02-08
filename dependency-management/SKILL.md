---
name: dependency-management
description: "Manage project dependencies, install packages, and handle version conflicts. Use when setting up projects, adding packages, updating dependencies, or resolving dependency issues."
---

# Dependency Management Skill

Manage project dependencies, install packages, and handle version conflicts.

## When to Use

Use this skill when the user wants to:
- Initialize a new project
- Add new dependencies
- Update existing dependencies
- Remove unused dependencies
- Resolve dependency conflicts
- Create lock files
- Manage package versions
- Set up CI/CD with dependencies

## Package Management

### npm (Node.js)
```bash
# Initialize project
npm init -y

# Install dependencies
npm install package-name

# Install dev dependencies
npm install --save-dev eslint prettier

# Install with specific versions
npm install package-name@1.0.0

# Install all dependencies
npm install

# Install as global
npm install -g package-name
```

### yarn (Node.js)
```bash
# Initialize project
yarn init -y

# Install dependencies
yarn add package-name

# Install dev dependencies
yarn add --dev eslint prettier

# Install all dependencies
yarn

# Install as global
yarn global add package-name
```

### pnpm (Node.js)
```bash
# Initialize project
pnpm init

# Install dependencies
pnpm add package-name

# Install dev dependencies
pnpm add -D eslint prettier

# Install all dependencies
pnpm install

# Install as global
pnpm add -g package-name

# Link local packages
pnpm link
```

### pip (Python)
```bash
# Initialize project
pip freeze > requirements.txt

# Install dependencies
pip install package-name

# Install from requirements
pip install -r requirements.txt

# Install in development mode
pip install -e .[dev]

# Freeze requirements
pip freeze > requirements.txt
```

### gem (Ruby)
```bash
# Initialize project
bundle init

# Install dependencies
gem install package-name

# Install from Gemfile
bundle install

# Install development dependencies
bundle install --development

# Add to Gemfile
bundle add package-name --group development
```

### cargo (Rust)
```bash
# Initialize project
cargo new project-name

# Add dependencies
cargo add package-name

# Add to Cargo.toml
cargo edit package-name

# Update dependencies
cargo update
cargo outdated
```

### composer (PHP)
```bash
# Initialize project
composer init

# Install dependencies
composer require package-name

# Install from composer.json
composer install

# Add as dev dependency
composer require --dev package-name

# Add to composer.json
composer require package-name --dev
```

## Version Management

### Semantic Versioning
```json
{
  "main": "1.0.0",
  "version": {
    "major": 1,
    "minor": 0,
    "patch": 0
  }
}
```

**Format**: `MAJOR.MINOR.PATCH`
- **MAJOR**: Breaking changes
- **MINOR**: Backward-compatible features
- **PATCH**: Bug fixes

### Package Management Files

#### package.json (npm)
```json
{
  "name": "my-project",
  "version": "1.0.0",
  "dependencies": {
    "lodash": "^4.17.21",
    "axios": "^0.27.2"
  },
  "devDependencies": {
    "eslint": "^8.0.0",
    "prettier": "^2.8.0"
  },
  "scripts": {
    "start": "node index.js",
    "test": "jest"
  }
}
```

#### package-lock.json (npm)
- Ensures consistent installs
- Locks exact versions

#### yarn.lock
- Alternative to package-lock.json
- Ensures consistent installs

#### requirements.txt (Python)
```
Django==4.2.0
Flask==2.3.0
numpy==1.24.0
pandas==2.0.0
```

#### Gemfile (Ruby)
```ruby
source 'https://rubygems.org'

gem 'rails', '~> 7.0'
gem 'jbuilder', '~> 2.11'

group :development, :test do
  gem 'rspec-rails', '~> 6.0'
end
```

#### Cargo.toml (Rust)
```toml
[package]
name = "my-project"
version = "0.1.0"
edition = "2021"

[dependencies]
serde = "1.0"
tokio = { version = "1", features = ["full"] }
```

#### composer.json (PHP)
```json
{
  "name": "php-project",
  "require": {
    "php": "^8.0",
    "ext-curl": "*",
    "ext-json": "*"
  },
  "require-dev": {
    "phpunit/phpunit": "^10.0"
  }
}
```

## Dependency Operations

### Adding Dependencies
```bash
# npm
npm install package-name

# yarn
yarn add package-name

# pnpm
pnpm add package-name
```

### Updating Dependencies
```bash
# Check for updates
npm outdated

# Update to latest
npm update

# Update specific package
npm update package-name

# Check security vulnerabilities
npm audit
npm audit fix
```

### Removing Dependencies
```bash
# Remove from package.json
npm uninstall package-name

# Remove from yarn.lock
yarn remove package-name

# Remove from pnpm
pnpm remove package-name
```

### Checking for Updates
```bash
# npm
npm outdated

# yarn
yarn outdated

# pnpm
pnpm outdated
```

### Security
```bash
# Audit for vulnerabilities
npm audit
yarn audit
pnpm audit

# Fix vulnerabilities
npm audit fix
yarn audit fix
pnpm audit fix
```

## Dependency Best Practices

### Keep Dependencies Current
- Regularly update dependencies
- Fix security vulnerabilities
- Review breaking changes

### Remove Unused Dependencies
```bash
# Find unused dependencies
npm prune

# yarn prune
yarn prune

# pnpm prune
pnpm prune
```

### Use Specific Versions
```json
{
  "dependencies": {
    "lodash": "^4.17.21"
  }
}
```
- `^4.17.21`: Compatible with 4.17.x
- `4.17.21`: Exact version
- `~4.17.21`: Minor compatible (4.17.x)

### Group Dependencies
```json
{
  "dependencies": {},
  "devDependencies": {}
}
```

### Lock Files
- Always commit lock files
- Ensure reproducible builds
- Handle version conflicts

### External vs Local
```json
{
  "dependencies": {
    "external-package": "^1.0.0"
  },
  "devDependencies": {
    "local-package": "file:../local-package"
  }
}
```

## CI/CD and Dependencies

### Cache Dependencies
```yaml
# CI configuration
- uses: actions/setup-node@v3
  with:
    node-version: '18'
    cache: 'npm'

# Run install once
- run: npm ci
- run: npm test
```

### Version Locking
```json
{
  "overrides": {
    "package-name": "1.0.0"
  }
}
```

## Large Projects

### Monorepo
```json
{
  "workspaces": ["packages/*"]
}
```

### Peer Dependencies
```json
{
  "peerDependencies": {
    "react": "^16.8.0"
  }
}
```

## Delivered Files

- package.json files
- requirements.txt files
- Gemfile files
- Cargo.toml files
- composer.json files
- Lock files
- Documentation

## Quality Checklist

- Dependencies are installed correctly
- Lock files are committed
- Security vulnerabilities are fixed
- Unused dependencies are removed
- Version ranges are appropriate
- External dependencies are minimal
- Local dependencies are properly linked
- Documentation is updated
