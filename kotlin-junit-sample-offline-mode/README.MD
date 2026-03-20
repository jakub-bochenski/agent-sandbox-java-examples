# Kotlin JUnit Sample - Offline Mode

This example demonstrates a stricter setup that prevents the agent from downloading any dependencies. Maven is forced to use only a project-scoped cache, with network access to Maven Central revoked.

## Configuration Applied

Extends [`kotlin-junit-sample-volume-persistence`](../kotlin-junit-sample-volume-persistence/README.MD) with project-scoped offline cache.

### 1. Project-Scoped Repository Created

Downloaded all project dependencies to a dedicated repository on the host:

```sh
./mvnw org.apache.maven.plugins:maven-dependency-plugin:3.6.1:go-offline \
  -Dmaven.repo.local="${HOME}/.config/agent-sandbox/my-project-repo"
```

This creates a Maven repository containing only the dependencies required by this project, separate from the global `~/.m2/repository`.

### 2. Volume Mount Updated

Modified `.agent-sandbox/compose/user.override.yml`:

```yaml
agent:
  build: .
  volumes:
    # Maven cache
    - m2-cache:/home/dev/.m2
    # Read-only host Maven Wrapper (caches Maven distributions)
    - ${HOME}/.m2/wrapper:/home/dev/.m2/wrapper:ro
    # Read-only host Maven cache
    - ${HOME}/.config/agent-sandbox/my-project-repo:/home/dev/.m2-host/repository:ro 
```

### 3. Maven Settings Configuration

Created Maven settings file inside the container (same as layered cache example):

```sh
agentbox compose exec -T agent sh -c 'cat > /home/dev/.m2/settings.xml' <<-'XML'
<settings>
    <profiles>
        <profile>
            <id>layered</id>

            <repositories>
                <repository>
                    <id>local-cache</id>
                    <url>file://${user.home}/.m2-host/repository</url>
                    <releases><enabled>true</enabled></releases>
                    <snapshots><enabled>true</enabled></snapshots>
                </repository>
                <repository>
                    <id>central</id>
                    <url>https://repo.maven.apache.org/maven2</url>
                </repository>
            </repositories>

            <pluginRepositories>
                <pluginRepository>
                    <id>local-cache</id>
                    <url>file://${user.home}/.m2-host/repository</url>
                </pluginRepository>
                <pluginRepository>
                    <id>central</id>
                    <url>https://repo.maven.apache.org/maven2</url>
                </pluginRepository>
            </pluginRepositories>
        </profile>
    </profiles>

    <activeProfiles>
        <activeProfile>layered</activeProfile>
    </activeProfiles>
</settings>
XML
```

### 4. Network Policy Restricted

Removed Maven Central from `.agent-sandbox/policy/user.policy.yaml`:

```yaml
services: []

domains: []
```


## How It Works

Maven attempts to resolve dependencies from the project-scoped cache at `file:///home/dev/.m2-host/repository`. If it tries to fall back to Maven Central, the request is blocked by the network policy.

This setup ensures:
- The agent cannot download new dependencies
- Only pre-approved project dependencies are available
- The cache contains only what this specific project needs, not the entire host's Maven history
- Network access is limited to the Claude API only

## Result

Builds succeed using only cached dependencies. Any attempt to add a new dependency or change versions will fail at the network boundary, making dependency changes explicit and auditable.

This is the strictest dependency isolation model, suitable for high-security environments where dependency provenance must be controlled.
