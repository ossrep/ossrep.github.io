# Microservices

Detailed documentation for each microservice in the OSSREP platform.

## Customer Service

**Repository:** `customer-service`

Manages customer accounts, contacts, and service locations.

### Responsibilities

- Customer account lifecycle (create, update, close)
- Contact information management
- Service address/premise management
- Customer preferences and communications

### Key Entities

- `Customer` - Core customer account
- `Contact` - Customer contact information
- `ServiceLocation` - Physical service address (ESI ID in Texas)
- `CustomerPreference` - Communication and billing preferences

---

## Billing Service

**Repository:** `billing-service`

Handles all billing, invoicing, and payment operations.

### Responsibilities

- Invoice generation and calculation
- Payment processing and allocation
- Account balance management
- Collections and dunning workflows

### Key Entities

- `Invoice` - Customer bill
- `InvoiceLine` - Line items on invoice
- `Payment` - Payment transaction
- `AccountBalance` - Current account state

---

## Usage Service

**Repository:** `usage-service`

Processes and manages electricity usage data.

### Responsibilities

- Receive and store meter readings
- Process interval data from Smart Meter Texas
- Calculate usage summaries
- Provide usage data for billing

### Key Entities

- `MeterReading` - Point-in-time meter read
- `IntervalData` - 15-minute interval usage
- `UsageSummary` - Aggregated usage for billing periods

---

## Enrollment Service

**Repository:** `enrollment-service`

Manages customer enrollment and ERCOT market transactions.

### Responsibilities

- New customer enrollment (switch requests)
- Move-in and move-out processing
- ERCOT transaction submission and tracking
- Enrollment status management

### Key Entities

- `EnrollmentRequest` - Customer enrollment/switch request
- `ErcotTransaction` - Transaction submitted to ERCOT
- `ServiceOrder` - Internal service order

---

## Product Service

**Repository:** `product-service`

Manages electricity products, rate plans, and pricing.

### Responsibilities

- Product catalog management
- Rate plan configuration
- Pricing calculation rules
- Contract terms and conditions

### Key Entities

- `Product` - Electricity product offering
- `RatePlan` - Pricing structure
- `PricingTier` - Tiered pricing levels
- `Contract` - Customer contract terms

---

## Gateway Service

**Repository:** `gateway-service`

API gateway for routing and cross-cutting concerns.

### Responsibilities

- Request routing to appropriate services
- Authentication and authorization
- Rate limiting
- Request/response logging
