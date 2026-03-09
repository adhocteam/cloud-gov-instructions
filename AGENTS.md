# AGENTS.md

## Autonomy tiers

Default to the lowest tier that accomplishes the task. Never skip tiers.

| Tier | Mode | What the agent does |
|------|------|---------------------|
| 0: OBSERVE | Read-only | Run read-only CF commands, analyze logs, review configurations |
| 1: RECOMMEND | Suggest only | Analyze and suggest changes without modifying anything |
| 2: DRAFT | Edit files | Create or modify local files (manifests, CI workflows, code) |
| 3: DEPLOY (non-prod) | Execute in dev/staging | Run modifying CF commands in non-production spaces after confirmation |
| 4: DEPLOY (prod, gated) | Execute in production | Requires explicit confirmation and impact statement before any modifying command |

## CF command safety

Always ask for explicit user confirmation before running any CF CLI command that modifies or destroys resources.

If the space name contains `prod`, `production`, or `prd`, state the org, space, app name, and impact before proceeding.

### Commands that always require confirmation

| Command | Risk | Impact |
|---------|------|--------|
| `cf delete` | HIGH | Permanently deletes an application |
| `cf delete-service` | HIGH | Permanently deletes a service and its data |
| `cf unbind-service` | MEDIUM | Disconnects app from service, may cause errors |
| `cf delete-service-key` | MEDIUM | Revokes external access credentials |
| `cf stop` | MEDIUM | Stops application, causes downtime |
| `cf delete-route` | MEDIUM | Removes route, breaks external access |
| `cf unmap-route` | MEDIUM | Removes route mapping from app |
| `cf push` | MEDIUM | Deploys code, may introduce bugs |
| `cf restage` | MEDIUM | Rebuilds app, causes brief downtime |
| `cf restart` | LOW | Restarts app, causes brief downtime |
| `cf scale` | LOW | Changes resources, may affect availability |
| `cf set-env` | LOW | Changes configuration, requires restart |
| `cf update-service` | LOW | Modifies service configuration |

### Commands safe to run without confirmation

`cf apps`, `cf app`, `cf services`, `cf service`, `cf logs --recent`, `cf env`, `cf routes`, `cf marketplace`, `cf target`, `cf buildpacks`

## Action audit trail

At the end of any session that involves CF commands or file modifications, summarize:

- Commands executed (every `cf` command, with org/space context)
- Files modified (with brief description)
- Recommendations made (not yet acted upon)
- Risks identified (warnings or concerns raised)

Format:

```
## Session Actions
- [EXECUTED] cf push my-app -f manifest-dev.yml (org: my-org, space: dev)
- [MODIFIED] manifest-dev.yml — added instances: 2 for availability
- [RECOMMENDED] Rotate service account credentials (last rotated 85 days ago)
- [WARNING] Production space detected — confirmed with user before proceeding
```

NIST 800-53: AU-2 (Audit Events), AU-3 (Content of Audit Records)

## NIST control references in code

When generating security-relevant code, include NIST SP 800-53 control references in comments using the format `NIST 800-53: <CONTROL-ID> - <CONTROL-NAME>`.

- Authentication code → IA family (IA-2, IA-5, IA-8)
- Logging/audit code → AU family (AU-2, AU-3, AU-6, AU-12)
- Input validation → SI family (SI-10, SI-11)
- Encryption/TLS → SC family (SC-8, SC-13, SC-28)
- Configuration files → CM family (CM-2, CM-6)
- CI/CD workflows → SA-10, CM-3

See `security.instructions.md` for detailed control mapping and code examples.
