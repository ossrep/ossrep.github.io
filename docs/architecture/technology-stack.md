# Technology Stack

OSSREP is built on Red Hat technologies and partners to provide enterprise-grade reliability and support.

## API Gateway & Service Mesh

### Red Hat Connectivity Link

API gateway solution providing:

- **External Gateway** - Public-facing APIs for customer portal and mobile apps
- **Internal Gateway** - Administrative APIs for the admin portal

Key capabilities:

| Capability          | Description                                      |
| ------------------- | ------------------------------------------------ |
| Traffic Management  | Rate limiting, load balancing, circuit breaking  |
| Security            | mTLS, API key management, OAuth integration      |
| Observability       | Request logging, metrics, distributed tracing    |
| Policy Enforcement  | Access control, request/response transformation  |

### Red Hat Service Mesh

Istio-based service mesh for secure service-to-service communication:

| Capability          | Description                                      |
| ------------------- | ------------------------------------------------ |
| mTLS                | Automatic encryption between services            |
| Traffic Control     | Canary deployments, A/B testing, fault injection |
| Observability       | Distributed tracing, service topology            |
| Resilience          | Retries, timeouts, circuit breakers              |

## Backend Services

### Java & Quarkus

All backend services are built using:

- **Java** - Latest LTS release
- **Quarkus** - Supersonic Subatomic Java framework
- **Maven** - Build and dependency management

Key Quarkus extensions used across services:

| Extension                       | Purpose                        |
| ------------------------------- | ------------------------------ |
| `rest-jackson`                  | REST API with JSON             |
| `hibernate-orm-panache`         | Database access                |
| `flyway`                        | Database migrations            |
| `smallrye-openapi`              | OpenAPI documentation          |
| `smallrye-health`               | Health checks                  |
| `micrometer-registry-prometheus`| Metrics                        |
| `oidc`                          | Authentication                 |
| `smallrye-reactive-messaging`   | Kafka integration              |

### Database

- **PostgreSQL** - Primary data store for all services
- Each service has its own database schema
- Flyway manages schema migrations

## Frontend

### Angular

- **Angular** - Latest LTS release
- **Bootstrap** - Styling via ng-bootstrap components
- **Signals** - Reactive state management
- OIDC integration for authentication

## Messaging

### Apache Kafka

- Event streaming platform (Confluent or AMQ Streams)
- Enables asynchronous communication between services
- Supports event sourcing and CQRS patterns

## Infrastructure

### Container Platform

- **OpenShift** - Enterprise Kubernetes platform
- **ROSA** - Red Hat OpenShift on AWS for cloud deployments

### Identity Management

- **Keycloak** - Reference OIDC provider
- Services are OIDC-agnostic for flexibility

### Automation

- **Ansible** - Infrastructure and deployment automation

## Serialization

### Protocol Buffers

- Used for efficient service-to-service communication
- Proto3 syntax
- Strict enum conventions with `UNSPECIFIED` as zero value
