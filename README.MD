# Agent Sandbox - Java/Kotlin Examples

This repository demonstrates running LLM coding agents in a sandboxed environment using [Agent Sandbox](https://github.com/mattolson/agent-sandbox). Each example builds on the previous one, showing progressively more sophisticated configurations for Java/Kotlin/Maven projects.

These examples focus on what's needed specifically for Java/Kotlin/Maven projects. The configurations handle Java-specific concerns like:
- Proxy configuration for Java tools (doesn't honor standard environment variables)
- Maven repository management and caching strategies
- Project-scoped dependency isolation


## Examples

### [kotlin-junit-sample-basic](./kotlin-junit-sample-basic/)

Basic sandbox setup with Java runtime and Maven support. Shows how to:
- Initialize Agent Sandbox for a Java project
- Add OpenJDK 17 to the container
- Configure network policy for Maven Central
- Set up Java proxy settings for HTTPS traffic

Start here if you're new to sandboxing Java projects.

### [kotlin-junit-sample-volume-persistence](./kotlin-junit-sample-volume-persistence/)

Adds Maven artifact caching with Docker volumes. Dependencies persist across container restarts, avoiding redundant downloads.

### [kotlin-junit-sample-layered-cache](./kotlin-junit-sample-layered-cache/)

Two-tier cache setup that mounts the host's Maven repository read-only. Dependencies already on the host are available immediately without network access. New artifacts go to the container-specific volume.

### [kotlin-junit-sample-offline-mode](./kotlin-junit-sample-offline-mode/)

Strictest isolation: network access to Maven Central is revoked entirely. Maven can only use a pre-populated project-scoped cache. Prevents the agent from downloading any new dependencies.

## Getting Started

Install Agent Sandbox following the [official documentation](https://github.com/mattolson/agent-sandbox?tab=readme-ov-file#quick-start-macos--colima).

Clone this repository and navigate to any example:

```bash
$ cd kotlin-junit-sample-basic
$ agentbox exec

dev@ab75dca99874:/workspace% claude --dangerously-skip-permissions # we are in a sandbox!
```

## What Agent Sandbox Provides

- **Filesystem and Environment isolation**: Only the project directory is mounted
- **Network filtering**: HTTP proxy enforces an allowlist, blocks everything else
- **Read-only config**: Policy files can't be modified by the agent
- **IDE integration**: Works with VS Code and JetBrains devcontainers (examples not included in this repo, but it's an add-on functionality)
