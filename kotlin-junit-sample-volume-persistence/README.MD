# Kotlin JUnit Sample - Volume Persistence

This example builds on the basic setup by adding Maven artifact caching with Docker volumes.

## Configuration Applied

Extends [`kotlin-junit-sample-basic`](../kotlin-junit-sample-basic/README.MD) with persistent Maven cache.

### 1. Docker Volume Added ([1de5358](/../../commit/1de5358))

Modified [`.agent-sandbox/compose/user.override.yml`](.agent-sandbox/compose/user.override.yml):

```yaml
agent:
  # ...
  volumes:
    - m2-cache:/home/dev/.m2 # this is the folder where Maven stores downloaded artifacts
# ...
volumes:
  m2-cache:
```

### 2. Dockerfile Updated ([1de5358](/../../commit/1de5358))

Added directory creation to [`Dockerfile`](.agent-sandbox/compose/Dockerfile):

```dockerfile
RUN mkdir -p /home/dev/.m2 # required for proper file ownership when mounting
```

Required for proper file ownership when the volume is mounted.

## Result

Maven dependencies persist across container restarts. First build downloads artifacts, later builds reuse the cache.

## Next

[Use host cache to speed up dependency downloadds](../kotlin-junit-sample-layered-cache/README.MD) or alternatively [Work in offline mode](../kotlin-junit-sample-offline-mode/README.MD)
