# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [1.2.0] - 2026-03-09

### Added
- Autonomy tiers (Tier 0-4) in AGENTS.md for structured agent operating modes
- Action audit trail guidance in AGENTS.md for FedRAMP AU-family compliance
- Automatic log redaction patterns in logging.instructions.md (AWS keys, GitHub tokens, JWTs, CF auth)
- Correlation ID guidance in logging.instructions.md using X-Vcap-Request-Id
- Pre-deployment validation chain in cicd.instructions.md with GitHub Actions example
- Deployment notification severity mapping in cicd.instructions.md
- Credential anti-pattern catalog in security.instructions.md
- Adversarial input awareness section in security.instructions.md for prompt injection resistance

### Changed
- Optimized AGENTS.md from 257 to 75 lines by removing content redundant with copilot-instructions.md and the six instruction files
- Merged destructive and resource-modifying command tables into a single table in AGENTS.md
- Compressed NIST control reference guidance into a concise mapping list

### Removed
- Agent Capabilities section from AGENTS.md (covered by instruction files)
- Context Awareness section from AGENTS.md (duplicated in copilot-instructions.md)
- Key Constraints and References sections from AGENTS.md (duplicated in copilot-instructions.md)
- Error Prevention Hierarchy from AGENTS.md (meta-guidance; specific rules already encode behavior)

## [1.1.0] - 2026-02-05

### Added
- cf-troubleshoot skill for diagnosing common cloud.gov/CF issues
- Updated README with skills documentation

### Changed
- Enhanced README structure and examples

## [1.0.0] - 2026-02-05

### Added
- Initial release with core instruction files
- Repository-level `.github/copilot-instructions.md`
- `AGENTS.md` with AI agent behaviors and safety guardrails
- Six core instruction files:
  - `deployment.instructions.md`
  - `manifest.instructions.md`
  - `services.instructions.md`
  - `cicd.instructions.md`
  - `security.instructions.md`
  - `logging.instructions.md`
- Compliance documentation agent
- MIT License
- Contributing guidelines
- Comprehensive README with quick start guide

[Unreleased]: https://github.com/adhocteam/cloud-gov-instructions/compare/v1.2.0...HEAD
[1.2.0]: https://github.com/adhocteam/cloud-gov-instructions/compare/v1.1.0...v1.2.0
[1.1.0]: https://github.com/adhocteam/cloud-gov-instructions/compare/v1.0.0...v1.1.0
[1.0.0]: https://github.com/adhocteam/cloud-gov-instructions/releases/tag/v1.0.0
