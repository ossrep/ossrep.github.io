# Architecture Overview

OSSREP follows a microservices architecture pattern, with each service responsible for a specific business domain.

## High-Level Architecture

```
    ┌──────────────────┐                    ┌──────────────────┐
    │ Customer Portal  │                    │  Admin Portal    │
    │   (Angular)      │                    │   (Angular)      │
    │                  │                    │                  │
    │   Mobile Apps    │                    │                  │
    └────────┬─────────┘                    └────────┬─────────┘
             │                                       │
             ▼                                       ▼
    ┌──────────────────┐                    ┌──────────────────┐
    │ External Gateway │                    │ Internal Gateway │
    │  (Connectivity   │                    │  (Connectivity   │
    │      Link)       │                    │      Link)       │
    └────────┬─────────┘                    └────────┬─────────┘
             │                                       │
             └───────────────┬───────────────────────┘
                             │
                             ▼
            ┌────────────────────────────────┐
            │       Red Hat Service Mesh     │
            │         (Istio-based)          │
            └────────────────┬───────────────┘
                             │
        ┌─────────────┬──────┴──────┬─────────────┐
        ▼             ▼             ▼             ▼
   ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐
   │Customer │  │ Meter   │  │ Billing │  │ Market  │  ...
   │ Service │  │ Service │  │ Service │  │ Service │
   └────┬────┘  └────┬────┘  └────┬────┘  └────┬────┘
        │            │            │            │
        ▼            ▼            ▼            ▼
   ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐
   │   DB    │  │   DB    │  │   DB    │  │   DB    │
   └─────────┘  └─────────┘  └─────────┘  └─────────┘
        │            │            │            │
        └────────────┴─────┬──────┴────────────┘
                           ▼
           ┌───────────────────────────┐
           │      Apache Kafka         │
           │   (Event Streaming)       │
           └───────────────────────────┘
```

## Gateway Architecture

OSSREP uses a dual-gateway pattern with Red Hat Connectivity Link to separate external and internal traffic:

### External Gateway

Handles all public-facing API traffic:

- **Customer Portal** - Self-service web application
- **Mobile Apps** - iOS and Android applications
- Public API consumers

Exposes only customer-facing endpoints with appropriate rate limiting and security policies.

### Internal Gateway

Handles all internal administrative traffic:

- **Admin Portal** - Internal operations and customer service
- Administrative APIs with elevated privileges

Restricted to internal network access with additional authentication requirements.

## Design Principles

### Domain-Driven Design

Each microservice is aligned with a specific bounded context. See [Domain Model](domain-model.md) for detailed entity relationships.

| Service  | Bounded Context                                        |
| -------- | ------------------------------------------------------ |
| Customer | Customers, accounts, premises, meters                  |
| Meter    | Meter readings, data validation, usage calculations    |
| Billing  | Invoices, payments, and collections                    |
| Market   | EDI transactions, market participant communications    |
| Product  | Rate plans, products, and pricing structures           |

#### Customer Hierarchy

The Customer Service supports a flexible hierarchy to accommodate both residential and commercial customers:

- **Individual customers**: Typically one account with one or more premises
- **Business customers**: Multiple accounts (by department/location), each with multiple premises and meters
- **Billing**: Occurs at the account level, with aggregated views for business customers

### Database per Service

Each service owns its data and has a dedicated PostgreSQL database. Services communicate through:

- **Synchronous** - REST APIs for queries and commands
- **Asynchronous** - Kafka events for eventual consistency

### Authentication & Authorization

- All services integrate with OIDC-compliant identity providers
- Keycloak serves as the reference implementation
- JWT tokens are used for service-to-service authentication
- Role-based access control (RBAC) at the API level

## Deployment

Services are containerized and deployed to OpenShift (Red Hat's enterprise Kubernetes distribution). Infrastructure automation is handled through Ansible.
