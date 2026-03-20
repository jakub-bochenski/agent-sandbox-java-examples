# Kotlin JUnit Sample - Basic Sandboxed Setup

This example shows the result of following the basic Agent Sandbox setup for a Java/Kotlin project.

## Configuration Applied

### 1. Sandbox Initialization

```bash
agentbox init
```

Selected Claude agent in CLI mode, which generated `.agent-sandbox/` containing the `docker compose` configuration and network allowlist.

### 2. Java Runtime

Created `Dockerfile`:

```dockerfile
FROM ghcr.io/mattolson/agent-sandbox-claude

USER root

RUN apt-get update && \
    apt-get install -y openjdk-17-jdk && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

USER dev
```

Modified `.agent-sandbox/compose/user.override.yml`:

```yaml
services:
  agent:
    image: !reset null
    build: .
```

### 3. Network Policy

Added Maven Central to `.agent-sandbox/policy/user.policy.yaml`:

```yaml
services: []

domains:
  - repo.maven.apache.org
```

### 4. Java Proxy Configuration

Added to the agent service in `.agent-sandbox/compose/user.override.yml`:

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
