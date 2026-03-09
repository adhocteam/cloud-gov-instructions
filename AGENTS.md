# Cloud.gov Agent Instructions

This document provides guidance for AI coding agents working with applications deployed to the cloud.gov platform.

## Agent Capabilities

When assisting with cloud.gov projects, you can help with:

### Deployment & Configuration
- Creating and modifying `manifest.yml` files
- Configuring buildpacks and runtime settings
- Setting up multi-environment deployments (dev, staging, prod)
- Writing `.cfignore` files to optimize deployments

### Service Integration
- Configuring database connections (PostgreSQL, MySQL)
- Setting up S3 bucket access for file storage
- Integrating Redis for caching and session management
- Working with Elasticsearch/OpenSearch for search functionality

### CI/CD Pipelines
- Creating GitHub Actions workflows for automated deployments
- Setting up service account authentication
- Implementing blue-green deployment strategies
- Configuring environment-specific variables and secrets

### Application Code
- Implementing health check endpoints
- Configuring applications to read `VCAP_SERVICES` credentials
- Setting up structured logging to stdout/stderr
- Following Twelve-Factor App principles

### Security & Compliance
- Implementing secrets management patterns
- Configuring egress restrictions
- Following FedRAMP compliance guidelines
- Setting up proper authentication flows

## Agent Behaviors

### Autonomy Tiers

Agents operate at defined autonomy tiers that determine what actions they can take. Start at the lowest tier appropriate for the task and only escalate when necessary.

| Tier | Mode | What the Agent Does | Examples |
|------|------|---------------------|----------|
| **0: OBSERVE** | Read-only | Run read-only CF commands, analyze logs, review configurations | `cf apps`, `cf logs --recent`, `cf env` |
| **1: RECOMMEND** | Suggest only | Analyze and suggest changes without modifying anything | "You should add `instances: 2` to your manifest" |
| **2: DRAFT** | Edit files | Create or modify local files (manifests, CI workflows, code) | Edit `manifest.yml`, create `.github/workflows/deploy.yml` |
| **3: DEPLOY (non-prod)** | Execute in dev/staging | Run modifying CF commands in non-production spaces after confirmation | `cf push` in dev or staging spaces |
| **4: DEPLOY (prod, gated)** | Execute in production | Run modifying CF commands in production — requires explicit confirmation and impact statement | `cf push --strategy rolling` in production |

**Tier Selection Rules:**
- **Default to the lowest tier** that accomplishes the task
- **Read-only questions** (app status, log analysis) → Tier 0
- **Code generation** (new manifests, CI pipelines) → Tier 2
- **Deployments** → Tier 3 (non-prod) or Tier 4 (prod), always with confirmation
- **Never skip tiers** — if the task requires Tier 4, verbally confirm the environment and impact before proceeding

### Critical Safety Rules

**ALWAYS ask for explicit user confirmation before running any CF CLI command that modifies or destroys resources.** These commands can cause service outages, data loss, or unintended costs.

#### Destructive Commands (ALWAYS require confirmation)

| Command | Risk | Impact |
|---------|------|--------|
| `cf delete` | **HIGH** | Permanently deletes an application |
| `cf delete-service` | **HIGH** | Permanently deletes a service and its data |
| `cf unbind-service` | **MEDIUM** | Disconnects app from service, may cause errors |
| `cf delete-service-key` | **MEDIUM** | Revokes external access credentials |
| `cf stop` | **MEDIUM** | Stops application, causes downtime |
| `cf delete-route` | **MEDIUM** | Removes route, breaks external access |
| `cf unmap-route` | **MEDIUM** | Removes route mapping from app |

#### Resource-Modifying Commands (Require confirmation in production)

| Command | Risk | Impact |
|---------|------|--------|
| `cf push` | **MEDIUM** | Deploys code, may introduce bugs |
| `cf restage` | **MEDIUM** | Rebuilds app, causes brief downtime |
| `cf restart` | **LOW** | Restarts app, causes brief downtime |
| `cf scale` | **LOW** | Changes resources, may affect availability |
| `cf set-env` | **LOW** | Changes configuration, requires restart |
| `cf update-service` | **LOW** | Modifies service configuration |

#### Safe Commands (No confirmation needed)

These read-only commands are safe to run without confirmation:
- `cf apps`, `cf app <name>`
- `cf services`, `cf service <name>`
- `cf logs <name> --recent`
- `cf env <name>`
- `cf routes`
- `cf marketplace`
- `cf target`
- `cf buildpacks`

### Environment Awareness

Before running any modifying command:

1. **Identify the target environment** - Check if targeting dev, staging, or production
2. **Warn about production** - If the space name contains `prod`, `production`, or `prd`, provide an explicit warning
3. **Confirm the org/space** - State which org and space will be affected
4. **Explain the impact** - Describe what the command will do before running it

Example confirmation request:
> "This will delete the application `my-app` in the **production** space (`org-name/prod`). This action is irreversible and will cause immediate downtime. Do you want me to proceed?"

### Error Prevention Hierarchy

When designing safety measures, prefer structural prevention over warnings. Apply this hierarchy from most effective to least:

| Level | Approach | Cloud.gov Example |
|-------|----------|-------------------|
| **1. Make Impossible** | Structural prevention — the system won't allow it | Agent cannot execute `cf delete-service` without parsing an explicit confirmation response |
| **2. Make Hard** | Extra steps + approval required | Production deploys require stating org, space, and app name explicitly before proceeding |
| **3. Make Visible** | Warnings + confirmations shown | Warning banner displayed when space name contains `prod`, `production`, or `prd` |
| **4. Make Reversible** | Easy rollback + audit trail | Blue-green deployments allow instant rollback via route swap; all actions logged |

Always prefer a higher level when possible. Don't rely on warnings (level 3) when you can make the mistake structurally impossible (level 1).

### Action Audit Trail

When performing CF operations or making recommendations, maintain an audit trail of significant actions taken during the session. This supports FedRAMP AU-family controls.

**NIST 800-53: AU-2 — Audit Events | AU-3 — Content of Audit Records**

At the end of any session that involves CF commands or file modifications, summarize:

1. **Commands executed** — Every `cf` command run, with org/space context
2. **Files modified** — Every file created or changed, with a brief description
3. **Recommendations made** — Suggestions given but not yet acted upon
4. **Risks identified** — Any warnings or concerns raised during the session

Example summary format:

```
## Session Actions
- [EXECUTED] cf push my-app -f manifest-dev.yml (org: my-org, space: dev)
- [MODIFIED] manifest-dev.yml — added instances: 2 for availability
- [MODIFIED] .github/workflows/deploy.yml — added health check verification step
- [RECOMMENDED] Rotate service account credentials (last rotated 85 days ago)
- [WARNING] Production space detected — confirmed with user before proceeding
```

This trail should be available for compliance review and incident analysis.

### When Generating Security-Relevant Code

**Always include NIST SP 800-53 control references** in code comments and documentation to support the ATO process:

1. **Document controls in docstrings/comments** - Reference specific control IDs (e.g., `NIST 800-53: IA-2`)
2. **Match controls to functionality** - Authentication → IA family, Logging → AU family, Input validation → SI family
3. **Use consistent format** - `NIST 800-53: <CONTROL-ID> - <CONTROL-NAME>`
4. **Include controls in config files** - Add control references as comments in manifests and workflows

See `security.instructions.md` for detailed control mapping and code examples.

### When Creating Deployment Configurations

1. **Always use explicit settings** - Don't rely on defaults for memory, instances, or buildpacks
2. **Include service bindings** - Reference all required services in manifest.yml
3. **Set appropriate instance counts** - Use 2+ instances for production workloads
4. **Configure health checks** - Include health check configuration for reliable deployments

### When Working with Services

1. **Never hardcode credentials** - Always read from `VCAP_SERVICES` environment variable
2. **Use service keys sparingly** - Prefer bound service credentials when possible
3. **Document service dependencies** - Keep manifest.yml in sync with required services

### When Writing CI/CD Pipelines

1. **Use secrets management** - Store CF credentials in GitHub Secrets, never in code
2. **Target specific spaces** - Always explicitly set org and space in automation
3. **Implement deployment verification** - Check app status after deployment
4. **Enable rollback capability** - Use blue-green deployments for zero-downtime updates

### When Modifying Application Code

1. **Stateless design** - Never rely on local filesystem for persistent data
2. **Graceful shutdown** - Handle SIGTERM signals for clean restarts
3. **Environment configuration** - Use environment variables for all settings
4. **Structured logging** - Output JSON logs to stdout for observability

## Context Awareness

### Recognizing Cloud.gov Projects

Look for these indicators that a project uses cloud.gov:

- `manifest.yml` or `manifest.*.yml` files in the repository
- References to `api.fr.cloud.gov` in configuration or scripts
- `VCAP_SERVICES` or `VCAP_APPLICATION` in code
- `.cfignore` file present
- GitHub Actions workflows with `cf` CLI commands

### Understanding the Environment

- **Production API**: `api.fr.cloud.gov`
- **Dashboard**: `dashboard.fr.cloud.gov`
- **Login**: `login.fr.cloud.gov`
- **Default app domain**: `app.cloud.gov`

### Common File Patterns

| Pattern | Purpose |
|---------|---------|
| `manifest.yml` | Primary deployment configuration |
| `manifest-*.yml` | Environment-specific manifests |
| `vars.yml`, `vars-*.yml` | Variable files for manifest interpolation |
| `.cfignore` | Deployment exclusions |
| `.profile` | Pre-start shell commands |
| `Procfile` | Process type definitions |

## Instruction Files

This repository includes additional instruction files in `.github/instructions/` for specific tasks:

- **deployment.instructions.md** - Detailed deployment workflows
- **services.instructions.md** - Service binding and configuration
- **manifest.instructions.md** - Manifest file structure and options
- **cicd.instructions.md** - CI/CD pipeline configuration
- **security.instructions.md** - Security and compliance guidance
- **logging.instructions.md** - Logging and monitoring setup

Refer to these files for detailed, task-specific guidance.

## Key Constraints

### Platform Limitations

- **Egress restrictions**: Outbound traffic may be limited - check with platform team
- **Memory limits**: Default quotas apply - request increases if needed
- **Buildpack availability**: Only certain buildpacks are available - check `cf buildpacks`
- **Container restarts**: Apps restart frequently - design for resilience

### Security Requirements

- **No secrets in code**: Use environment variables or bound services
- **HTTPS only**: All external traffic must use HTTPS
- **Compliance documentation**: Maintain awareness of FedRAMP shared responsibility

### Operational Expectations

- **Log to stdout**: Platform captures stdout/stderr automatically
- **Quick startup**: Apps should start within 60 seconds
- **Health endpoints**: Implement health checks for platform monitoring

## References

- [cloud.gov Documentation](https://docs.cloud.gov/)
- [Cloud Foundry Documentation](https://docs.cloudfoundry.org/)
- [Twelve-Factor App](https://12factor.net/)
- [cloud.gov Status Page](https://cloudgov.statuspage.io/)
