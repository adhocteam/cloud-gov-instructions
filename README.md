# Cloud.gov Copilot Instructions

A reusable set of instruction files for AI coding assistants (GitHub Copilot) that encapsulate cloud.gov platform guidance and best practices.

## Overview

This repository provides a comprehensive set of instruction files that help AI coding assistants understand and follow cloud.gov conventions. Use these files as a starting point for any project deployed to the cloud.gov platform.

### What's Included

| File | Purpose |
|:------|:---------|
| `.github/copilot-instructions.md` | Repository-level cloud.gov platform overview |
| `AGENTS.md` | AI agent behaviors and safety guardrails |
| `.github/instructions/` | Task-specific instruction files |
| `.github/agents/` | Custom Copilot agents for specialized tasks |

## Quick Start

### 1. Copy Files to Your Repository

Copy the following files to your cloud.gov project:

```bash
# Clone this repository
git clone https://github.com/your-org/cloud-gov-instructions.git

# Copy instruction files to your project
cp -r cloud-gov-instructions/.github/copilot-instructions.md your-project/.github/
cp -r cloud-gov-instructions/.github/instructions your-project/.github/
cp -r cloud-gov-instructions/.github/agents your-project/.github/
cp cloud-gov-instructions/AGENTS.md your-project/
```

### 2. Customize for Your Project

Update the copied files as needed:

1. **`.github/copilot-instructions.md`**: Add project-specific context
2. **`AGENTS.md`**: Adjust agent behaviors for your team's needs
3. **Instruction files**: Modify patterns or add custom guidance

### 3. Start Developing

GitHub Copilot will automatically use these instructions when:

- Editing files that match `applyTo` patterns (e.g., `manifest*.yml`)
- Working with cloud.gov-related code
- Running custom agents for compliance documentation

## Instruction Files Reference

### Core Instructions

| File | Applies To | Description |
|:------|:-----------|:-------------|
| [deployment.instructions.md](.github/instructions/deployment.instructions.md) | `manifest*.yml`, `Procfile`, `.cfignore`, `.profile` | Deployment workflows, blue-green deployments, scaling |
| [manifest.instructions.md](.github/instructions/manifest.instructions.md) | `manifest*.yml`, `vars*.yml` | Manifest configuration, properties reference |
| [services.instructions.md](.github/instructions/services.instructions.md) | `*.py`, `*.js`, `*.ts`, `*.rb`, `*.java`, `*.go`, `manifest*.yml` | RDS, S3, Redis service integration |

### DevOps & Security

| File | Applies To | Description |
|:------|:-----------|:-------------|
| [cicd.instructions.md](.github/instructions/cicd.instructions.md) | `.github/workflows/*.yml`, `Jenkinsfile`, `.circleci/config.yml` | CI/CD pipelines, service accounts |
| [security.instructions.md](.github/instructions/security.instructions.md) | `*.py`, `*.js`, `*.ts`, `*.rb`, `*.java`, `*.go`, `manifest*.yml` | FedRAMP compliance, secrets management, NIST controls |
| [logging.instructions.md](.github/instructions/logging.instructions.md) | `*.py`, `*.js`, `*.ts`, `*.rb`, `*.java`, `*.go`, `manifest*.yml` | Structured logging, log drains |

### Custom Agents

| Agent | Purpose |
|:-------|:---------|
| [compliance-docs.agent.md](.github/agents/compliance-docs.agent.md) | Generate SSP sections and Control Implementation Matrices |

#### Invoking Custom Agents in VS Code

To use the compliance documentation agent in VS Code:

1. Open the Copilot Chat panel (`Ctrl+Shift+I` / `Cmd+Shift+I`)
2. Type `@` to see available agents, then select `compliance-docs`
3. Enter your prompt, for example:

```
@compliance-docs Generate a Control Implementation Matrix for this project
```

Or use specific prompts like:

```
@compliance-docs Scan the codebase for NIST control references and create SSP sections
```

```
@compliance-docs Document which controls are inherited from cloud.gov vs implemented by our application
```

## Key Features

### AI Safety Guardrails

The `AGENTS.md` file includes critical safety rules that prevent AI assistants from running destructive Cloud Foundry commands without explicit confirmation:

- **Destructive commands** (HIGH risk): `cf delete`, `cf delete-service` - Always require confirmation
- **Modifying commands** (MEDIUM risk): `cf push`, `cf restage` - Require confirmation in production
- **Safe commands**: `cf apps`, `cf logs` - No confirmation needed

### NIST Control Documentation

When generating security-relevant code, AI assistants will include NIST SP 800-53 control references:

```python
def authenticate_user(username: str, password: str) -> User:
    """
    Authenticate user with credentials.
    
    NIST 800-53: IA-2 (Identification and Authentication)
    NIST 800-53: IA-5 (Authenticator Management)
    """
    # Implementation
```

### Compliance Documentation Generation

Use the compliance docs agent to generate:
- Control Implementation Matrix (spreadsheet format)
- System Security Plan sections
- Control inheritance documentation

## Project Structure

```
your-project/
├── .github/
│   ├── copilot-instructions.md    # Repository-level instructions
│   ├── instructions/
│   │   ├── deployment.instructions.md
│   │   ├── manifest.instructions.md
│   │   ├── services.instructions.md
│   │   ├── cicd.instructions.md
│   │   ├── security.instructions.md
│   │   └── logging.instructions.md
│   └── agents/
│       └── compliance-docs.agent.md
├── AGENTS.md                       # Agent behaviors and safety rules
├── manifest.yml                    # Your app's deployment config
└── ... (your application code)
```

## Requirements

- **GitHub Copilot**: Active subscription with custom instructions support
- **VS Code**: With GitHub Copilot extension installed
- **Cloud Foundry CLI**: For deployment commands (`cf`)

## Customization

### Adding Project-Specific Instructions

Create additional instruction files for your project:

```yaml
---
applyTo: "**/your-pattern/**"
---

# Your Custom Instructions

Instructions specific to your project...
```

### Extending Agent Behaviors

Add custom agent rules to `AGENTS.md`:

```markdown
### Project-Specific Rules

- Your custom rule here
- Another project-specific behavior
```

## FedRAMP Compliance

These instructions support FedRAMP compliance by:

1. **Documenting inherited controls** from cloud.gov
2. **Guiding implementation** of application-level controls
3. **Including NIST references** in generated code
4. **Generating compliance documentation** via custom agents

cloud.gov is FedRAMP Moderate authorized (Package ID: F1607067912), providing approximately 60% of required controls.

## Contributing

Contributions are welcome! Please:

1. Fork this repository
2. Create a feature branch
3. Submit a pull request with your improvements

## Resources

- [cloud.gov Documentation](https://docs.cloud.gov/)
- [Cloud Foundry Documentation](https://docs.cloudfoundry.org/)
- [GitHub Copilot Custom Instructions](https://docs.github.com/en/copilot/customizing-copilot/adding-custom-instructions-for-github-copilot)
- [NIST SP 800-53](https://csrc.nist.gov/publications/detail/sp/800-53/rev-5/final)
- [FedRAMP](https://www.fedramp.gov/)

## License

This project is licensed under the [MIT License](LICENSE).

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

Use at your own risk.
