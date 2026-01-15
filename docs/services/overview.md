# Services Overview

OSSREP is composed of several microservices, each responsible for a specific business domain within retail electricity operations.

## Core Services

### Customer Service

Manages customer lifecycle, accounts, premises, and meters. Supports both individual (residential) and business (commercial) customers with flexible account hierarchies.

**Repository:** `customer-service`

**[Full Documentation →](customer-service.md)**

**Key Capabilities:**

- Customer enrollment and onboarding (individual and business)
- 6-step enrollment flow with meter selection at the premise
- Multi-account support for business customers
- Premise management
- Meter registration and tracking
- Customer preferences and communication settings
- Aggregated views across accounts for business customers

---

### Meter Service

Handles meter assets and consumption data.

**Repository:** `meter-service`

**Key Capabilities:**

- Meter registration and tracking
- Meter reading collection
- Usage data validation (VEE)
- Interval data management
- Meter exchange tracking

---

### Billing Service

Generates invoices and manages payments.

**Repository:** `billing-service`

**Key Capabilities:**

- Invoice generation
- Payment processing
- Payment arrangements
- Collections management
- Account adjustments

---

### Market Service

Handles market transactions and regulatory communications.

**Repository:** `market-service`

**Key Capabilities:**

- EDI transaction processing
- Market participant messaging
- Enrollment transactions
- Usage request/response
- Regulatory reporting

---

### Product Service

Manages rate plans and product offerings.

**Repository:** `product-service`

**Key Capabilities:**

- Rate plan definitions
- Product catalog
- Pricing structures
- Promotional offers
- Contract terms

## Service Communication

Services communicate through two primary mechanisms:

### REST APIs

- Synchronous request/response
- OpenAPI documented
- JWT authentication

### Kafka Events

- Asynchronous event streaming
- Domain events for cross-service consistency
- Event schemas defined with Protocol Buffers

## Repository Structure

Each service repository follows a standard structure:

```
service-name/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── io/github/ossrep/servicename/
│   │   │       ├── api/           # REST resources
│   │   │       ├── service/       # Business logic
│   │   │       └── repository/    # Data access
│   │   └── resources/
│   │       ├── application.yaml
│   │       ├── db/migration/      # Flyway scripts
│   │       └── ValidationMessages.properties
│   └── test/
├── pom.xml
└── README.md
```
