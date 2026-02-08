---
name: devops
description: "Set up CI/CD pipelines, infrastructure as code, and deployment workflows. Use when creating pipelines, configuring automation, or implementing DevOps practices."
---

# DevOps Skill

Implement CI/CD pipelines, infrastructure automation, and DevOps workflows.

## When to Use

Use this skill when the user wants to:
- Create CI/CD pipelines
- Configure automation scripts
- Set up infrastructure as code
- Implement deployment workflows
- Work with GitOps practices
- Configure build and deployment tools
- Set up release management

## CI/CD Tools

### Pipeline Orchestration
- **GitHub Actions**: Git-based workflows
- **GitLab CI/CD**: GitLab integrated
- **Jenkins**: Classic CI server
- **CircleCI**: Cloud-based pipelines

### Infrastructure as Code
- **Terraform**: Multi-provider IaC
- **Ansible**: Configuration management
- **Puppet**: Configuration management
- **Chef**: Configuration management

### GitOps
- **ArgoCD**: GitOps controller
- **Flux**: GitOps toolkit
- **Sealed Secrets**: Git-based secrets

## Pipeline Stages

1. **Build**: Compile, package, test
2. **Test**: Unit, integration, E2E tests
3. **Deploy**: Staging, production
4. **Verify**: Smoke tests, monitoring
5. **Rollback**: Automatic rollback on failure

## Common Patterns

### Monorepo vs Multirepo
- **Monorepo**: Single repo, shared tooling
- **Multirepo**: Separate repos, independent releases

### Build Artifacts
- Docker images
- S3 artifacts
- Registry storage
- Version tagging

### Deployment Strategies
- Blue-green deployment
- Canary release
- Rolling update
- Shadow deployment

## Best Practices

- **Pipeline as code**: All in version control
- **Idempotent actions**: Safe to run multiple times
- **Caching**: Speed up builds with caches
- **Parallel steps**: Run independent tasks in parallel
- **Visibility**: Pipeline logs and status
- **Security**: Scan for vulnerabilities, secrets management

## Deliverables

- Pipeline configuration
- CI/CD scripts
- Documentation
- Rollback procedures
- Monitoring integration

## Quality Checklist

- Pipelines are automated and reliable
- Tests are run in pipeline
- Artifacts are versioned
- Rollback is possible
- Security scanning included
- Documentation is updated
- Pipeline is reusable
