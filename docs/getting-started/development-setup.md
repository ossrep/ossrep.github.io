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

For local development, you'll need PostgreSQL, Keycloak (two instances), and Kafka. The easiest way to start all services is with podman-compose.

#### Red Hat Registry Authentication

The compose file uses Red Hat container images. Authenticate with the registry first (requires a free Red Hat developer account):

```bash
podman login registry.redhat.io
```

#### Starting Services

```bash
cd ossrep.github.io
podman-compose up -d
```

This starts:

| Service | Image | Port | Purpose |
|---------|-------|------|---------|
| PostgreSQL | rhel9/postgresql-16:latest | 5432 | Database |
| Keycloak External | rhbk/keycloak-rhel9:26.0 | 8180 | Customers, partners |
| Keycloak Internal | rhbk/keycloak-rhel9:26.0 | 8181 | Admins, employees |
| Kafka | amq-streams/kafka-37-rhel9:2.7.0 | 9094 | Event streaming |

To stop all services:

```bash
podman-compose down
```

### Test Users

Both Keycloak instances come pre-configured with test users:

| Keycloak | Realm | Username | Password | Role |
|----------|-------|----------|----------|------|
| External (8180) | ossrep | alice | password | customer |
| External (8180) | ossrep | bob | password | customer |
| Internal (8181) | ossrep-internal | jsmith | password | csr |
| Internal (8181) | ossrep-internal | admin | password | admin |

See [Identity & Access Management](../architecture/identity.md) for full details.

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
