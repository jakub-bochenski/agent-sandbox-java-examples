FROM ghcr.io/mattolson/agent-sandbox-claude@sha256:ee8aed230787a816f0944a312b10a587adcc4a9553f0588a68ffc9e5fa43b238

USER root

RUN apt-get update && \
    apt-get install -y openjdk-17-jdk && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

USER dev

RUN mkdir -p /home/dev/.m2 # required for proper file ownership when mounting
