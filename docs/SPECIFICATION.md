# Cloud.gov Instructions Project Specification

## Project Overview

This project provides a reusable set of instruction files for AI coding agents working with applications deployed to the cloud.gov platform. The goal is to encapsulate guidance and best practices from cloud.gov documentation into structured instruction files that any project can adopt as a starting point.

## Objectives

1. Create comprehensive instruction files that guide AI agents in cloud.gov development workflows
2. Encapsulate cloud.gov best practices and conventions
3. Provide a reusable template that projects can customize for their specific needs
4. Reduce onboarding time for new projects deploying to cloud.gov

## Target Audience

- AI coding agents (GitHub Copilot, Claude, etc.) assisting developers
- Development teams starting new projects on cloud.gov
- Teams migrating existing applications to cloud.gov

## File Structure

### Primary Files

| File | Purpose |
|------|---------|
| `.github/copilot-instructions.md` | Repository-level custom instructions for GitHub Copilot |
| `AGENTS.md` | Agent-specific guidance and capabilities (root level) |
| `.github/instructions/*.instructions.md` | Task-specific instruction files |

### Instruction File Categories

The `.github/instructions/` directory will contain specialized instruction files for:

1. **Deployment** - CF CLI commands, manifest configuration, deployment strategies
2. **Services** - Binding and managing cloud.gov services (databases, S3, etc.)
3. **Authentication** - cloud.gov identity provider integration, UAA configuration
4. **Networking** - Routes, domains, CDN configuration
5. **Security** - Compliance, scanning, security best practices
6. **Monitoring** - Logging, metrics, health checks
7. **CI/CD** - GitHub Actions workflows for cloud.gov deployments

## Content Requirements

### Each Instruction File Must Include

- Clear scope definition using `applyTo` patterns where applicable
- Step-by-step guidance for common tasks
- Code examples and templates
- Error handling and troubleshooting guidance
- Links to relevant cloud.gov documentation

### Cloud.gov Topics to Cover

#### Core Platform Concepts
- [x] Organizations and spaces
- [x] User roles and permissions
- [x] Quotas and resource limits

#### Deployment
- [x] `manifest.yml` configuration
- [x] Buildpacks (standard and custom)
- [x] Docker container deployment
- [x] Blue-green deployments
- [x] Zero-downtime deployments

#### Services
- [x] Marketplace services
- [x] User-provided services
- [x] Service keys and credentials
- [x] Database services (PostgreSQL, MySQL)
- [x] S3 storage
- [ ] Elasticsearch/OpenSearch
- [x] Redis

#### Networking
- [x] Custom domains
- [x] Route management
- [ ] CDN integration
- [ ] Internal routes

#### Security & Compliance
- [x] FedRAMP compliance considerations
- [ ] Container scanning
- [x] Egress restrictions
- [x] Secrets management

#### Operations
- [x] Logging with `cf logs`
- [x] Application scaling
- [x] Health monitoring
- [ ] Incident response

#### CI/CD Integration
- [x] GitHub Actions workflows
- [x] Service account authentication
- [x] Automated deployments
- [x] Environment promotion strategies

## Instruction File Format

### copilot-instructions.md Structure

```markdown
# Cloud.gov Development Instructions

## Platform Overview
[Brief description of cloud.gov]

## Key Conventions
[Project-wide conventions and standards]

## Common Commands
[Frequently used CF CLI commands]

## Best Practices
[General best practices for cloud.gov development]
```

### Task-Specific Instruction File Structure

```markdown
---
applyTo: [glob pattern]
---

# [Task Name] Instructions

## Overview
[Brief description of the task]

## Prerequisites
[Required setup or knowledge]

## Steps
[Detailed step-by-step guidance]

## Examples
[Code examples and templates]

## Troubleshooting
[Common issues and solutions]

## References
[Links to cloud.gov documentation]
```

## Implementation Phases

### Phase 1: Foundation ✅
- [x] Create base `copilot-instructions.md`
- [x] Create `AGENTS.md` with agent capabilities
- [x] Set up `.github/instructions/` directory structure

### Phase 2: Core Instructions ✅
- [x] Deployment instructions
- [x] Service binding instructions
- [x] Manifest configuration guidance

### Phase 3: Advanced Topics ✅
- [x] CI/CD workflow templates
- [x] Security and compliance guidance
- [x] Monitoring and logging instructions

### Phase 4: Refinement ✅
- [x] Add troubleshooting sections
- [x] Include real-world examples
- [x] Create quick-start guide

### Additional Enhancements ✅
- [x] NIST SP 800-53 control documentation guidance
- [x] AI safety guardrails for destructive CF CLI commands
- [x] Custom compliance documentation agent

## Success Criteria

1. A new project can copy these instruction files and immediately have useful AI agent guidance
2. Instructions accurately reflect current cloud.gov documentation and best practices
3. File structure follows GitHub Copilot custom instructions conventions
4. Instructions are clear, actionable, and reduce common deployment errors

## References

- [cloud.gov Documentation](https://docs.cloud.gov/)
- [GitHub Copilot Repository Instructions](https://docs.github.com/en/copilot/how-tos/configure-custom-instructions/add-repository-instructions)
- [Cloud Foundry Documentation](https://docs.cloudfoundry.org/)

## Maintenance

These instruction files should be reviewed and updated:
- When cloud.gov releases significant platform updates
- When GitHub Copilot custom instructions format changes
- Quarterly, to ensure accuracy and relevance
