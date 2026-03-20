# Kotlin JUnit Sample - Layered Cache

This example builds on volume persistence by adding a read-only mount of the host's Maven repository. This creates a two-tier cache: dependencies already on the host are available immediately, newly downloaded artifacts go to the container-specific volume.

## Configuration Applied

Extends [`kotlin-junit-sample-volume-persistence`](../kotlin-junit-sample-volume-persistence/README.MD) with layered caching.

### 1. Host Repository Mount

Modified [`.agent-sandbox/compose/user.override.yml`](.agent-sandbox/compose/user.override.yml):

```yaml
agent:
  build: .
  volumes:
    # Maven cache
    - m2-cache:/home/dev/.m2
    # Read-only host Maven cache
    - ${HOME}/.m2/repository:/home/dev/.m2-host/repository:ro
```

The mount is read-only to prevent sandbox escape. We mount only the `repository/` subdirectory, not the entire `.m2/` directory, since the parent might contain credentials.

### 2. Maven Settings Configuration

Created Maven settings file inside the container to configure repository resolution order:

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

The settings file persists in the `m2-cache` volume defined earlier.

## How It Works

Maven checks repositories in order:
1. First checks `file:///home/dev/.m2-host/repository` (read-only mount of host cache)
2. If artifact not found, downloads from `https://repo.maven.apache.org/maven2`
3. Downloaded artifacts are stored in `/home/dev/.m2` (the writable volume)

This approach:
- Reuses artifacts already on the host (no network access needed)
- Isolates new downloads to the container volume
- Prevents the agent from modifying the host's Maven cache
- Avoids exposing credentials that might exist in `~/.m2/settings.xml`

## Result

Example Maven output showing local cache hit:

```
dev@3304d0309f88:/workspace% ./mvnw -DskipTests
Picked up JAVA_TOOL_OPTIONS: -Dhttp.proxyHost=proxy -Dhttp.proxyPort=8080 -Dhttps.proxyHost=proxy -Dhttps.proxyPort=8080 -Daether.connector.http.useSystemProperties=true
[INFO] Scanning for projects...
Downloading from local-cache: file:///home/dev/.m2-host/repository/org/jetbrains/kotlin/kotlin-maven-plugin/2.2.21/kotlin-maven-plugin-2.2.21.pom
```

Dependencies present on the host are resolved instantly from the filesystem, avoiding redundant downloads through the proxy.
