# Architecture Overview

OSSREP follows a microservices architecture pattern, with each service responsible for a specific business domain within the retail electric provider operations.

## High-Level Architecture

```mermaid
graph TB
    subgraph "Frontend Applications"
        CP[Customer Portal]
        AP[Admin Portal]
    end

    subgraph "API Gateway"
        GW[Gateway Service]
    end

    subgraph "Core Services"
        CS[Customer Service]
        BS[Billing Service]
        US[Usage Service]
        ES[Enrollment Service]
        PS[Product Service]
    end

    subgraph "Integration Services"
        ERCOT[ERCOT Integration]
        SMT[Smart Meter Texas]
        PAY[Payment Gateway]
    end

    subgraph "Infrastructure"
        DB[(PostgreSQL)]
        MQ[Message Queue]
    end

    CP --> GW
    AP --> GW
    GW --> CS
    GW --> BS
    GW --> US
    GW --> ES
    GW --> PS

    CS --> DB
    BS --> DB
    US --> DB
    ES --> DB
    PS --> DB

    ES --> ERCOT
    US --> SMT
    BS --> PAY

    CS <--> MQ
    BS <--> MQ
    US <--> MQ
    ES <--> MQ
```

## Design Principles

### Domain-Driven Design

Each microservice represents a bounded context aligned with a specific business domain:

- **Customer Context** - Customer accounts, contacts, service locations
- **Billing Context** - Invoices, payments, collections
- **Usage Context** - Meter data, consumption tracking
- **Enrollment Context** - Service requests, switches, move-ins/outs
- **Product Context** - Rate plans, pricing, contracts

### Event-Driven Architecture

Services communicate asynchronously through events for loose coupling:

- `CustomerCreated`, `CustomerUpdated`
- `UsageReceived`, `UsageProcessed`
- `InvoiceGenerated`, `PaymentReceived`
- `EnrollmentSubmitted`, `EnrollmentConfirmed`

### API-First Design

All services expose RESTful APIs following OpenAPI 3.0 specification.

## Repository Structure

| Repository | Description |
|------------|-------------|
| `ossrep.github.io` | Documentation site |
| `customer-service` | Customer management microservice |
| `billing-service` | Billing and invoicing microservice |
| `usage-service` | Usage data processing microservice |
| `enrollment-service` | Enrollment and ERCOT transactions |
| `product-service` | Products and rate plans |
| `gateway-service` | API gateway |
| `customer-portal` | Angular customer-facing application |
| `admin-portal` | Angular internal operations application |
| `ossrep-commons` | Shared libraries and utilities |
