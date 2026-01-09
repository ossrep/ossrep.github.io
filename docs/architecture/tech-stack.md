# Technology Stack

## Backend - Quarkus

[Quarkus](https://quarkus.io/) is a Kubernetes-native Java framework tailored for GraalVM and HotSpot.

### Why Quarkus?

- **Fast startup** - Critical for serverless and containerized deployments
- **Low memory footprint** - Efficient resource utilization
- **Developer experience** - Live reload, unified configuration
- **Cloud-native** - Built for Kubernetes, excellent container support
- **Reactive support** - Both imperative and reactive programming models
- **Extensions ecosystem** - Rich library of integrations

### Key Extensions Used

| Extension | Purpose |
|-----------|---------|
| `quarkus-resteasy-reactive` | RESTful API endpoints |
| `quarkus-hibernate-orm-panache` | Database ORM with active record pattern |
| `quarkus-jdbc-postgresql` | PostgreSQL connectivity |
| `quarkus-smallrye-openapi` | OpenAPI documentation |
| `quarkus-smallrye-jwt` | JWT authentication |
| `quarkus-smallrye-reactive-messaging` | Event-driven messaging |
| `quarkus-container-image-docker` | Container image building |

---

## Frontend - Angular

[Angular](https://angular.io/) is a platform for building web applications.

### Why Angular?

- **Enterprise-ready** - Mature, well-supported framework
- **TypeScript** - Strong typing for large codebases
- **Comprehensive** - Routing, forms, HTTP, testing built-in
- **Tooling** - Excellent CLI and IDE support
- **Component architecture** - Reusable, testable components

### Key Libraries

| Library | Purpose |
|---------|---------|
| `@angular/material` | UI component library |
| `@ngrx/store` | State management |
| `@angular/router` | Application routing |

---

## Database - PostgreSQL

[PostgreSQL](https://www.postgresql.org/) is a powerful open-source relational database.

### Why PostgreSQL?

- **Reliability** - ACID compliant, battle-tested
- **Features** - JSON support, full-text search, extensions
- **Performance** - Excellent query optimizer
- **Open source** - No licensing costs
- **Ecosystem** - Wide tooling and hosting support

### Database Strategy

- Each microservice owns its database schema
- No direct cross-service database access
- Flyway for schema migrations

---

## Infrastructure

### Containerization

- **Docker** - Container runtime
- **Kubernetes** - Container orchestration (production)
- **Docker Compose** - Local development

### CI/CD

- **GitHub Actions** - Automated builds, tests, deployments
- **GitHub Container Registry** - Container image storage

### Messaging

- **Apache Kafka** or **RabbitMQ** - Event streaming/messaging between services

### Observability

- **OpenTelemetry** - Distributed tracing
- **Prometheus** - Metrics collection
- **Grafana** - Dashboards and visualization
