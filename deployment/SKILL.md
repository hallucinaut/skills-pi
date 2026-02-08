---
name: deployment
description: "Deploy applications to production environments. Use when configuring deployments, creating deployment scripts, managing release processes, or setting up CI/CD pipelines."
---

# Deployment Skill

Deploy applications to production environments.

## When to Use

Use this skill when the user wants to:
- Configure deployment configurations
- Create deployment scripts
- Set up CI/CD pipelines
- Manage deployment strategies
- Handle release management
- Configure environments
- Deploy to cloud providers
- Implement blue-green deployments

## Deployment Types

### Local Deployment
```bash
# Start application
npm start
python app.py
```

### Development Deployment
```bash
# Local server with hot reload
npm run dev
nodemon app.js
```

### Staging Deployment
```bash
# Deploy to staging environment
npm run deploy:staging
```

### Production Deployment
```bash
# Deploy to production
npm run deploy:production
```

## Deployment Configurations

### Environment Variables
```javascript
// .env files
NODE_ENV=production
DATABASE_URL=postgresql://user:pass@localhost/db
API_KEY=your-api-key
PORT=3000
```

### Configuration Files

#### Environment-based Config
```javascript
// config.js
const config = {
  development: {
    database: 'dev_db',
    port: 3000
  },
  staging: {
    database: 'staging_db',
    port: 3000
  },
  production: {
    database: 'prod_db',
    port: process.env.PORT || 3000
  }
};

module.exports = config[process.env.NODE_ENV];
```

#### Docker Compose
```yaml
version: '3.8'
services:
  app:
    image: myapp:latest
    environment:
      - NODE_ENV=production
      - DATABASE_URL=${DATABASE_URL}
    ports:
      - "3000:3000"
    restart: always
```

### Deployment Scripts

#### Deployment Script
```bash
#!/bin/bash
set -e

# Build
npm run build

# Test
npm run test

# Deploy
echo "Deploying to production..."
scp dist/* server:/var/www/app/
ssh server "pm2 restart app"
```

#### NPM Scripts
```json
{
  "scripts": {
    "dev": "NODE_ENV=development nodemon app.js",
    "build": "npm run clean && npm run bundle",
    "start": "NODE_ENV=production node dist/app.js",
    "deploy:staging": "npm run build && scp -r dist/* staging-server:/var/www/app/",
    "deploy:production": "npm run build && scp -r dist/* production-server:/var/www/app/"
  }
}
```

## Deployment Strategies

### Rolling Deployment
```bash
# Update one instance at a time
docker pull myapp:latest
docker stop $(docker ps -q --filter ancestor=myapp:previous)
docker run -d -p 80:80 myapp:latest
```

### Blue-Green Deployment
```bash
# Blue environment
docker run -d -p 8080:80 myapp:1.0.0

# Green environment
docker run -d -p 80:80 myapp:1.0.0

# Switch traffic
nginx -s reload
```

### Canary Deployment
```bash
# Deploy to subset of servers
for i in {1..5}; do
  docker run -d -p 808$i:80 myapp:1.0.0
done

# Monitor metrics
# Gradually shift traffic
```

### Rolling Update
```bash
# Docker rolling update
docker service update --replicas=5 myapp

# Kubernetes rolling update
kubectl set image deployment/myapp myapp=myapp:1.0.0
```

## Cloud Deployment

### AWS Deployment
```bash
# Deploy with AWS CLI
aws deploy create-deployment \
  --application-name myapp \
  --deployment-group-name myapp-prod \
  --s3-location s3Bucket=my-s3-bucket,s3Key=app.zip
```

### Docker Deployment
```bash
# Build and push
docker build -t myapp:1.0.0 .
docker tag myapp:1.0.0 registry.example.com/myapp:1.0.0
docker push registry.example.com/myapp:1.0.0

# Deploy
docker run -d -p 80:80 \
  --name myapp \
  --restart unless-stopped \
  registry.example.com/myapp:1.0.0
```

### Kubernetes Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myapp:1.0.0
        ports:
        - containerPort: 80
```

## Release Management

### Version Bump
```bash
# Major version
npm version major

# Minor version
npm version minor

# Patch version
npm version patch

# Manual version
npm version 2.0.0
```

### Changelog
```markdown
# Changelog

## [1.0.0] - 2024-02-08
### Added
- Initial deployment
- Authentication system
- User dashboard

### Fixed
- Login bug
- Database connection
```

### Pre-release
```bash
# Pre-release version
npm version prerelease --preid=beta
```

## Post-deployment

### Health Checks
```javascript
// Health check endpoint
app.get('/health', (req, res) => {
  const health = {
    status: 'ok',
    uptime: process.uptime(),
    timestamp: Date.now()
  };
  res.json(health);
});
```

### Monitoring
```bash
# Monitor application
pm2 logs myapp
pm2 monit
pm2 status
```

### Rollback
```bash
# Rollback to previous version
docker stop $(docker ps -q -f name=myapp)
docker rm $(docker ps -q -f name=myapp)
docker pull registry.example.com/myapp:previous
docker run -d -p 80:80 myapp:previous
```

## CI/CD Integration

### GitHub Actions
```yaml
name: Deploy
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      - run: npm ci
      - run: npm run build
      - run: npm test
      - name: Deploy
        env:
          DEPLOY_KEY: ${{ secrets.DEPLOY_KEY }}
        run: ./deploy.sh
```

### GitLab CI
```yaml
deploy_production:
  stage: deploy
  script:
    - npm run build
    - scp dist/* production-server:/var/www/app/
  only:
    - main
```

## Environment Setup

### Configuration Files
```json
// .env.example
NODE_ENV=production
DATABASE_URL=postgresql://localhost:5432/myapp
REDIS_URL=redis://localhost:6379
```

### Database Migrations
```bash
# Apply migrations
npm run migrate:up
npm run migrate:down

# Seed database
npm run db:seed
```

## Rollback Procedures

### Quick Rollback
```bash
# Rollback deployment
./scripts/rollback.sh
```

### Full Rollback
```bash
# Restore from backup
./scripts/restore-backup.sh
```

## Deployment Documentation

### Deployment Checklist
- Build completed successfully
- Tests passed
- Environment variables configured
- Database migrations applied
- Health checks passing
- Monitoring is active
- Rollback plan is ready

### Deployment Logs
```bash
# View deployment logs
tail -f logs/deployment.log

# Deployment status
cat .deployment-status
```

## Deliverables

- Deployment configuration files
- Deployment scripts
- Environment documentation
- Rollback procedures
- Monitoring setup
- CI/CD configuration
- Deployment documentation

## Quality Checklist

- Deployments are automated
- Environment variables are configured
- Health checks are in place
- Rollback procedures documented
- Monitoring is set up
- Deployment logs are accessible
- Environment is consistent
- Documentation is updated
