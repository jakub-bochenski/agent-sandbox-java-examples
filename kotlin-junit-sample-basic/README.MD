# Kotlin JUnit Sample - Basic Sandboxed Setup

> [!NOTE]
> While file paths are shown for demonstration purposes, it's best to use `agentbox edit` to edit files.

This example shows the result of following the basic Agent Sandbox setup for a Java/Kotlin project.

## Configuration Applied

### 1. Sandbox Initialization ([d2fa7db](/../../commit/d2fa7db))

```bash
agentbox init
```

Selected Claude agent in CLI mode, which generated `.agent-sandbox/` containing the `docker compose` configuration and network allowlist.

### 2. Java Runtime ([056b07c](/../../commit/056b07c))

Created [`Dockerfile`](.agent-sandbox/compose/Dockerfile):

```dockerfile
FROM ghcr.io/mattolson/agent-sandbox-claude

USER root

RUN apt-get update && \
    apt-get install -y openjdk-17-jdk && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

USER dev
```

Modified [`.agent-sandbox/compose/user.override.yml`](.agent-sandbox/compose/user.override.yml):

```yaml
services:
  agent:
    image: !reset null
    build: .
```

### 3. Network Policy ([77f0bc7](/../../commit/77f0bc7))

Added Maven Central to [`.agent-sandbox/policy/user.policy.yaml`](.agent-sandbox/policy/user.policy.yaml):

```yaml
services: []

domains:
  - repo.maven.apache.org
```

### 4. Java Proxy Configuration ([a6d9337](/../../commit/a6d9337))

Added to the agent service in [`.agent-sandbox/compose/user.override.yml`](.agent-sandbox/compose/user.override.yml):

```yaml
environment:
  JAVA_TOOL_OPTIONS: -Dhttp.proxyHost=proxy -Dhttp.proxyPort=8080 -Dhttps.proxyHost=proxy -Dhttps.proxyPort=8080 -Daether.connector.http.useSystemProperties=true
```

Required because Java doesn't honor standard proxy environment variables.

## Result

The sandbox configuration allows:
- Building and testing the Kotlin project with Maven
- Downloading dependencies from Maven Central through the proxy
- Running the agent with file access limited to the project directory
- Network access restricted to Claude API and Maven Central

## Next
Dependencies are downloaded each time you restart the container.
See [kotlin-junit-sample-volume-persistence](../kotlin-junit-sample-volume-persistence/README.MD)
