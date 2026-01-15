# Services Overview

OSSREP is composed of several microservices, each responsible for a specific business domain within retail energy operations.

## Core Services

### Customer Service

Manages customer lifecycle and account information.

**Repository:** `customer-service`

**Key Capabilities:**

- Customer enrollment and onboarding
- Account management
- Contact information
- Service agreements
- Customer preferences

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
