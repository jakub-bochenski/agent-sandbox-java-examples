# Kotlin JUnit Sample - Devcontainer Mode

This example shows the initial state after running `agentbox init` with devcontainer mode. It demonstrates what files are created by the initialization process without any additional configuration.

## What `agentbox init` Created

Running this command:

```bash
agentbox init --agent claude --mode devcontainer --ide vscode
```

Generated the following structure:

```
.agent-sandbox/
├── compose/
│   ├── base.yml                           # Base Docker Compose configuration
│   ├── agent.claude.yml                   # Claude agent service definition
│   ├── mode.devcontainer.yml              # Devcontainer mode configuration
│   ├── user.override.yml                  # User-specific compose overrides
│   └── user.agent.claude.override.yml     # User-specific Claude overrides
├── policy/
│   ├── policy.devcontainer.yaml           # Base devcontainer policy
│   ├── user.policy.yaml                   # User-specific policy additions
│   └── user.agent.claude.policy.yaml      # Claude-specific policy additions
└── active-target.env                      # Active configuration target

.devcontainer/
├── devcontainer.json                      # VS Code/JetBrains devcontainer config
└── devcontainer.user.json                 # User-specific devcontainer settings
```

## What This Provides

Out of the box, this configuration:
- Isolates the agent to the project workspace only
- Blocks all network access except to Claude API endpoints
- Provides IDE integration through devcontainer support
- Mounts policy files read-only so the agent cannot modify its own permissions

## Next Steps

This is a starting point. The configuration can be extended with:
- Language runtimes (Java, Node.js, Python, etc.)
- Build tool configurations (Maven, Gradle, npm)
- Dependency caching strategies
- Additional network allowlists for package managers

See other examples in this repository for common extensions:
- [`kotlin-junit-sample-basic`](../kotlin-junit-sample-basic) - Adds Java runtime and Maven support
- [`kotlin-junit-sample-volume-persistence`](../kotlin-junit-sample-volume-persistence) - Adds artifact caching
- [`kotlin-junit-sample-layered-cache`](../kotlin-junit-sample-layered-cache) - Leverages host Maven repository
- [`kotlin-junit-sample-offline-mode`](../kotlin-junit-sample-offline-mode) - Strict offline dependency isolation
