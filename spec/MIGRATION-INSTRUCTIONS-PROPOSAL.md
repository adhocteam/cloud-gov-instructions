# Proposal: Migration Instructions File

## Overview

Create a new instruction file (`migration.instructions.md`) to help AI coding assistants support teams migrating applications to or from cloud.gov.

## Purpose

The current instruction set is optimized for development *within* cloud.gov. A migration-focused instruction file would extend coverage to teams:

- Moving applications from cloud.gov to another platform (Kubernetes, serverless, etc.)
- Migrating legacy applications *to* cloud.gov
- Evaluating platform portability of existing codebases

## Proposed Capabilities

### 1. Platform-Specific Pattern Detection

Help AI assistants identify code tightly coupled to Cloud Foundry:

- `VCAP_SERVICES` credential parsing
- CF-specific environment variables (`VCAP_APPLICATION`, `CF_INSTANCE_*`)
- Buildpack-dependent configurations
- `.profile` startup scripts
- Manifest-defined resource allocations

### 2. Concept Mapping Table

Provide translation guidance between CF concepts and common target platforms:

| Cloud Foundry | Kubernetes | Serverless (Fargate/Cloud Run) |
|---------------|------------|-------------------------------|
| `manifest.yml` | Deployment + Service YAML | Task definition / service config |
| `VCAP_SERVICES` | Secrets + ConfigMaps | Secrets Manager / env vars |
| Buildpacks | Dockerfile / Cloud Native Buildpacks | Container image |
| `cf scale` | HPA / replica count | Auto-scaling config |
| Health check endpoint | Liveness/readiness probes | Health check config |

### 3. Migration Artifact Generation

Guide AI assistants in generating:

- `Dockerfile` based on detected buildpack and runtime
- Environment variable templates from bound services
- Container orchestration configs (Kubernetes YAML, ECS task definitions)
- CI/CD pipeline updates for new deployment targets

### 4. Compliance Transition Checklist

Document security/compliance considerations when changing platforms:

- Inherited controls that require new implementation
- FedRAMP boundary changes
- Logging and audit trail reconfiguration
- Network security and egress rule updates

## Benefits

1. **Broader utility** — Makes the instruction set valuable for teams in transition, not just steady-state development
2. **Reduced lock-in concerns** — Demonstrates that platform investment doesn't create barriers to future changes
3. **Alignment with blog post themes** — Reinforces the "platforms reduce cognitive load" message by extending it to migration scenarios
4. **Community contribution opportunity** — Migration patterns vary by target platform; this creates a natural extension point for contributors

## Scope Considerations

The initial version should focus on:

- **Outbound migration** from cloud.gov (higher immediate value)
- **Common targets**: Kubernetes, AWS Fargate, Google Cloud Run
- **Pattern detection** over full automation

Future iterations could add:

- Inbound migration guidance (to cloud.gov)
- Platform-specific detailed instructions (e.g., `migration-to-kubernetes.instructions.md`)
- Automated migration script generation

## Estimated Effort

Medium — Requires research into portable patterns and target platform conventions, but follows established instruction file structure.
