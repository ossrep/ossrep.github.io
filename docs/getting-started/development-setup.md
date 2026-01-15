# Development Setup

This guide covers setting up your local development environment for OSSREP.

## Prerequisites

### System Requirements

- Fedora or RHEL-based Linux distribution
- 16GB RAM minimum (32GB recommended)
- 50GB available disk space

### Required Software

Install the following tools:

```bash
# Java (Latest LTS)
sudo dnf install java-21-openjdk-devel

# Maven
sudo dnf install maven

# Node.js (Latest LTS)
sudo dnf install nodejs

# Angular CLI
npm install -g @angular/cli

# Quarkus CLI
curl -Ls https://sh.jbang.dev | bash -s - trust add https://repo1.maven.org/maven2/io/quarkus/quarkus-cli/
curl -Ls https://sh.jbang.dev | bash -s - app install --fresh --force quarkus@quarkusio

# Podman (container runtime)
sudo dnf install podman podman-compose
```

### Local Services

For local development, you'll need:

```bash
# PostgreSQL
podman run -d --name postgres \
  -e POSTGRES_PASSWORD=postgres \
  -p 5432:5432 \
  postgres:latest

# Keycloak
podman run -d --name keycloak \
  -e KEYCLOAK_ADMIN=admin \
  -e KEYCLOAK_ADMIN_PASSWORD=admin \
  -p 8180:8080 \
  quay.io/keycloak/keycloak:latest start-dev

# Kafka (using Redpanda for simplicity)
podman run -d --name redpanda \
  -p 9092:9092 \
  docker.redpanda.com/redpandadata/redpanda:latest \
  redpanda start --smp 1 --memory 1G --overprovisioned
```

## Project Structure

Clone repositories into the organization folder:

```bash
cd ~/projects/github/ossrep

# Documentation site
git clone git@github.com:ossrep/ossrep.github.io.git

# Services (as they are created)
git clone git@github.com:ossrep/customer-service.git
git clone git@github.com:ossrep/meter-service.git
# ... etc
```

## Creating a New Quarkus Service

```bash
GROUP_ID=io.github.ossrep
ARTIFACT_ID=customer-service

quarkus create app --wrapper --no-code \
    -x config-yaml,rest-jackson,smallrye-openapi,smallrye-health,micrometer-registry-prometheus \
    $GROUP_ID:$ARTIFACT_ID:0.0.1-SNAPSHOT

cd "$ARTIFACT_ID"
git init
git add .
git commit -m 'Quarkus project init'
```

## Creating a New Angular Application

```bash
NG_PROJECT_NAME=customer-portal

ng new --routing --style scss --ssr false --zoneless false --defaults $NG_PROJECT_NAME

cd "$NG_PROJECT_NAME"
ng add @ng-bootstrap/ng-bootstrap --skip-confirmation
ng generate environments
git add .
git commit -m 'Angular project init'
```

## Running Services

### Quarkus Dev Mode

```bash
cd customer-service
./mvnw quarkus:dev
```

The service will be available at `http://localhost:8080` with:

- OpenAPI UI: `http://localhost:8080/q/swagger-ui`
- Health: `http://localhost:8080/q/health`
- Metrics: `http://localhost:8080/q/metrics`

### Angular Dev Server

```bash
cd customer-portal
ng serve
```

The application will be available at `http://localhost:4200`.

## IDE Setup

### IntelliJ IDEA / JetBrains

- Install the Quarkus plugin
- Enable annotation processing for Lombok (if used)

### VS Code / Cursor

Recommended extensions:

- Quarkus
- Java Extension Pack
- Angular Language Service
