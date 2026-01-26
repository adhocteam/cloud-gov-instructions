# Contributing to Cloud.gov Copilot Instructions

Thank you for your interest in contributing! This project provides reusable AI coding assistant instructions for the cloud.gov platform, and we welcome contributions that improve the quality and coverage of these instructions.

## How to Contribute

### Reporting Issues

If you find an issue with the instructions—whether it's outdated information, unclear guidance, or missing coverage—please open a GitHub issue. Include:

- A clear description of the problem
- The specific instruction file(s) affected
- Any relevant cloud.gov documentation links
- Suggested improvements (if you have them)

### Submitting Changes

1. **Fork the repository** and create a feature branch from `main`
2. **Make your changes** following the patterns established in existing instruction files
3. **Test your changes** by using the instructions in a real project if possible
4. **Submit a pull request** with a clear description of what you've changed and why

### What We're Looking For

Contributions that:

- Keep instruction files accurate and up-to-date with cloud.gov documentation
- Add coverage for cloud.gov features not yet documented
- Improve clarity and reduce ambiguity in existing instructions
- Add helpful examples or troubleshooting guidance
- Fix typos, broken links, or formatting issues

### Instruction File Guidelines

When creating or modifying instruction files:

- Follow the structure established in existing files
- Include the appropriate `applyTo` frontmatter for context-aware activation
- Reference official cloud.gov documentation where applicable
- Include NIST control references for security-relevant guidance
- Keep instructions actionable and specific

### Agent and Skill Guidelines

When modifying `AGENTS.md`, custom agents, or skills:

- Maintain the safety guardrails for destructive CF CLI commands
- Document any new agent behaviors clearly
- Test agent behaviors in VS Code before submitting

## Questions?

If you have questions about contributing, feel free to open an issue for discussion.
