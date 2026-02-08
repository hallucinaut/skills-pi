# Skills for Pi

A collection of custom skills for the Pi coding agent to enhance its capabilities.

## Skills

### Core Skills
- **frontend-design** - Design and implement distinctive, production-ready frontend interfaces
- **backend-api** - Build RESTful APIs, GraphQL endpoints, and database-driven backend services
- **database-schema** - Design and document database schemas, migrations, relationships, and constraints
- **security-audit** - Perform security assessments, vulnerability scanning, and penetration testing

### Development Skills
- **testing** - Write unit tests, integration tests, and end-to-end tests
- **docker** - Containerize applications with Docker and Docker Compose
- **kubernetes** - Deploy and manage Kubernetes applications
- **devops** - Set up CI/CD pipelines, infrastructure as code, and deployment workflows
- **scripting** - Write automation scripts, command-line tools, and utility programs
- **authentication** - Implement authentication and authorization systems
- **file-operations** - Perform file system operations and manage files and directories
- **validation** - Validate input data and ensure data integrity

### Quality Skills
- **code-review** - Analyze code quality, security vulnerabilities, performance issues, and best practices
- **linter** - Apply code linting, formatting, and style checks
- **performance-optimization** - Optimize application performance, reduce latency, and improve resource usage
- **debugging** - Diagnose and fix bugs efficiently

### Observability Skills
- **monitoring** - Set up monitoring, logging, and observability
- **documentation** - Write and maintain clear, comprehensive documentation

### Infrastructure Skills
- **cloud** - Manage cloud resources across AWS, GCP, or Azure
- **dependency-management** - Manage project dependencies and package management

### Workflow Skills
- **git-workflow** - Manage git operations, branching strategies, and version control workflows
- **deployment** - Deploy applications to production environments

## Installation

To use these skills with Pi:

```bash
# Clone this repository
git clone https://github.com/hallucinaut/skills-pi.git ~/custom/agent/skills/
rm -rf ~/custom/agent/skills/.git; rm -rf ~/custom/agent/skills/README.md

# Add to your Pi settings
# Create or edit ~/.pi/settings.json and add:
# {
#   "skills": ["~/custom/agent/skills"]
# }
```

## License

MIT License
