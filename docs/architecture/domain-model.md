# Domain Model

This document describes the core domain model for OSSREP, focusing on the customer hierarchy and relationships between key entities.

## Customer Hierarchy Overview

OSSREP supports two types of customers with different organizational structures:

```
Customer
  └── Accounts (1+ for business, typically 1 for individual)
        └── Premises (1+)
              └── Meters (1+)
```

## Customer Types

### Individual Customers

Individual customers are residential consumers. They typically have a simpler account structure:

- **Typical case:** Single account, single premise, single meter
- **Extended case:** Single account with multiple premises (e.g., primary residence + vacation home or rental property)
- Each premise may have multiple meters (e.g., multiple electric meters for different circuits or buildings)

```
Individual Customer (John Smith)
  └── Account #1234567890
        ├── Premise: 123 Main Street, Anytown, TX (Primary)
        │     └── Electric Meter: E-12345678
        │
        └── Premise: 456 Beach Blvd, Galveston, TX (Vacation Home)
              └── Electric Meter: E-23456789
```

### Business Customers

Business customers have a more complex organizational structure to support enterprise needs:

- **Multiple accounts:** Businesses may have separate accounts for different departments, cost centers, or organizational units
- **Multiple premises:** Each account can have multiple physical locations
- **Multiple meters:** Each location can have multiple meters

```
Business Customer (Acme Corporation)
  ├── Account: Headquarters (#9876543210)
  │     └── 100 Corporate Drive, Dallas, TX
  │           └── Electric Meter: E-34567890
  │
  ├── Account: Warehouse District (#5555555555)
  │     ├── 200 Industrial Pkwy, Building A, Fort Worth, TX
  │     │     └── Electric Meter: E-45678901
  │     └── 200 Industrial Pkwy, Building B, Fort Worth, TX
  │           └── Electric Meter: E-56789012
  │
  └── Account: Retail Locations (#7777777777)
        ├── 500 Shopping Center Blvd, Store 101, Plano, TX
        │     └── Electric Meter: E-67890123
        └── 750 Mall Drive, Suite 220, Arlington, TX
              └── Electric Meter: E-78901234
```

## Core Entities

### Customer

The top-level entity representing an individual or business that has a relationship with the utility.

| Attribute | Type | Description |
|-----------|------|-------------|
| `customerId` | Long | Unique identifier |
| `type` | Enum | `individual` or `business` |
| `firstName` | String | First name (individual only) |
| `lastName` | String | Last name (individual only) |
| `businessName` | String | Business name (business only) |
| `createdAt` | Timestamp | Customer creation date |
| `status` | Enum | `active`, `inactive`, `pending` |

### Account

A billing entity that groups premises. Billing occurs at the account level.

| Attribute | Type | Description |
|-----------|------|-------------|
| `accountId` | Long | Unique identifier |
| `accountNumber` | String | Human-readable account number |
| `customerId` | Long | Reference to parent customer |
| `nickname` | String | Optional friendly name (e.g., "Headquarters") |
| `status` | Enum | `active`, `inactive`, `pending` |
| `currentBalance` | Decimal | Current amount due |
| `createdAt` | Timestamp | Account creation date |

!!! info "Billing Level"
    All billing operations (invoices, payments, collections) occur at the **account level**. This allows businesses to receive separate bills for different organizational units while maintaining a single customer relationship.

### Premise

A physical location where utility services are provided.

| Attribute | Type | Description |
|-----------|------|-------------|
| `premiseId` | Long | Unique identifier |
| `accountId` | Long | Reference to parent account |
| `street` | String | Street address |
| `unit` | String | Apartment/unit number (optional) |
| `city` | String | City |
| `state` | String | State code |
| `zip` | String | ZIP/postal code |
| `isPrimary` | Boolean | Primary premise for the account |
| `serviceStartDate` | Date | When service began at this location |

### Meter

A physical device that measures electricity consumption at a premise.

| Attribute | Type | Description |
|-----------|------|-------------|
| `meterId` | Long | Unique identifier |
| `meterNumber` | String | Physical meter identifier |
| `premiseId` | Long | Reference to premise |
| `type` | Enum | `electric` |
| `status` | Enum | `available`, `active`, `inactive` |
| `installDate` | Date | Meter installation date |
| `lastReading` | Decimal | Most recent meter reading |
| `lastReadingDate` | Date | Date of most recent reading |
| `unit` | String | Unit of measure (`kWh`) |

!!! info "Meter Selection During Enrollment"
    During the enrollment process, customers select which meters at a premise to enroll for service. Meters with status `available` are shown to new customers. Once enrolled, the meter status changes to `active`.

## Product Entities

### Plan

A rate plan or product offering that determines pricing and contract terms.

| Attribute | Type | Description |
|-----------|------|-------------|
| `planId` | Long | Unique identifier |
| `name` | String | Plan display name |
| `description` | String | Marketing description |
| `ratePerKwh` | Decimal | Energy rate in cents per kWh |
| `termMonths` | Integer | Contract term length in months |
| `earlyTerminationFee` | Decimal | Fee for early cancellation |
| `baseCharge` | Decimal | Monthly base/service charge |
| `renewablePercent` | Integer | Percentage of renewable energy (0-100) |
| `planType` | Enum | `fixed`, `variable`, `indexed` |
| `features` | List | Marketing features/benefits |
| `status` | Enum | `active`, `inactive`, `archived` |

### Plan Documents

Each plan requires regulatory disclosure documents:

| Document | Description |
|----------|-------------|
| **EFL** | Electricity Facts Label - standardized pricing disclosure required by PUCT |
| **TOS** | Terms of Service - contract terms and conditions |
| **YRAC** | Your Rights as a Customer - consumer rights information |

!!! info "Plan Selection During Enrollment"
    After entering a service address, customers choose from available plans for their ZIP code. The selected plan determines their energy rate, contract term, and any early termination fees.

### Contract

A contract represents the formal agreement between the customer and the REP for electricity service under a specific plan.

| Attribute | Type | Description |
|-----------|------|-------------|
| `contractId` | Long | Unique identifier |
| `contractNumber` | String | Human-readable contract number |
| `customerId` | Long | Reference to customer |
| `planId` | Long | Reference to the selected plan |
| `startDate` | Date | Service start date |
| `endDate` | Date | Contract end date (calculated from plan term) |
| `status` | Enum | `pending`, `active`, `expired`, `cancelled`, `renewed` |
| `signedAt` | Timestamp | When customer accepted terms |
| `earlyTerminationFee` | Decimal | ETF amount if cancelled early (from plan) |
| `createdAt` | Timestamp | Contract creation date |

### Contract-Account Association

A contract can cover one or more accounts. This allows business customers to have different plans for different accounts, or a single contract covering multiple accounts.

| Attribute | Type | Description |
|-----------|------|-------------|
| `contractId` | Long | Reference to contract |
| `accountId` | Long | Reference to account |
| `addedAt` | Timestamp | When account was added to contract |

```
Contract
  ├── Plan (determines rate and terms)
  └── Accounts (1+)
        └── Premises
              └── Meters
```

**Contract Lifecycle:**

1. **Pending** - Created during enrollment, awaiting service start date
2. **Active** - Service has started, contract is in effect
3. **Expired** - Contract term has ended (customer may renew or switch)
4. **Cancelled** - Customer terminated early (ETF may apply)
5. **Renewed** - Customer renewed into a new contract

!!! info "Contract vs Plan"
    A **Plan** is a product offering with defined rates and terms. A **Contract** is the specific agreement with a customer, including the actual start/end dates and which accounts are covered. Multiple customers can be on the same plan, but each has their own contract.

## Billing Entities

### Bill

An invoice generated for an account covering a billing period.

| Attribute | Type | Description |
|-----------|------|-------------|
| `billId` | Long | Unique identifier |
| `accountId` | Long | Reference to account |
| `amount` | Decimal | Total amount due |
| `dueDate` | Date | Payment due date |
| `status` | Enum | `pending`, `unpaid`, `paid`, `overdue` |
| `billingPeriodStart` | Date | Start of billing period |
| `billingPeriodEnd` | Date | End of billing period |
| `paidDate` | Date | Date payment was received (if paid) |
| `paidAmount` | Decimal | Amount paid (if paid) |

## Entity Relationships

```
┌──────────────────────────────────────────────────────────────────┐
│                           CUSTOMER                                │
│  (Individual or Business)                                        │
└──────────────────────────────────────────────────────────────────┘
                │                                       │
                │ 1:N                                   │ 1:N
                ▼                                       ▼
┌─────────────────────────────┐         ┌─────────────────────────────┐
│          ACCOUNT            │◄────────│         CONTRACT            │
│  - Billing entity           │   N:M   │  - Plan reference           │
│  - Receives invoices        │         │  - Start/end dates          │
│  - Processes payments       │         │  - Terms acceptance         │
└─────────────────────────────┘         └──────────────┬──────────────┘
        │           │                                  │
        │ 1:N       │ 1:N                              │ N:1
        ▼           ▼                                  ▼
┌───────────────┐  ┌───────────────┐         ┌─────────────────────────────┐
│   PREMISE     │  │     BILL      │         │           PLAN              │
│  - Location   │  │  - Invoice    │         │  - Rate per kWh             │
│  - Address    │  │  - Payment    │         │  - Term months              │
└───────┬───────┘  └───────────────┘         │  - Features                 │
        │                                    └─────────────────────────────┘
        │ 1:N
        ▼
┌───────────────┐
│    METER      │
│  - Readings   │
│  - Usage      │
└───────────────┘
```

**Key Relationships:**

- A **Customer** has one or more **Accounts**
- A **Customer** has one or more **Contracts** (over time)
- A **Contract** references one **Plan** and covers one or more **Accounts**
- An **Account** has one or more **Premises** and **Bills**
- A **Premise** has one or more **Meters**

## Aggregation Rules

### Dashboard Aggregation

For customer-facing dashboards, data is aggregated across all accounts:

| Metric | Aggregation |
|--------|-------------|
| Total Balance Due | Sum of all unpaid bills across all accounts |
| Total Accounts | Count of active accounts |
| Total Premises | Count across all accounts |
| Total Meters | Count across all premises |
| Usage by Type | Sum of usage grouped by meter type |

### Business Customer View

Business customers see an **aggregated view** of all their accounts by default:

- Single dashboard showing totals across all accounts
- Ability to drill down into individual accounts
- Account-level filtering for bills and usage

### Individual Customer View

Individual customers see a **simplified view** since they typically have one account:

- Direct display of account information
- Premise list showing all locations
- Meter details for each location

## API Considerations

### Customer Context

All API operations are performed within the context of the authenticated customer. The customer ID is derived from the JWT token claims.

```
GET /api/accounts                    → Returns all accounts for the authenticated customer
GET /api/accounts/{accountId}        → Returns specific account (must belong to customer)
GET /api/bills                       → Returns bills across all customer accounts
GET /api/usage                       → Returns usage across all customer meters
```

### Account Scope

Some operations can be scoped to a specific account:

```
GET /api/accounts/{accountId}/premises
GET /api/accounts/{accountId}/bills
GET /api/accounts/{accountId}/usage
```

### Meter Operations

Meter operations are typically accessed through the premise:

```
GET /api/premises/{premiseId}/meters
GET /api/meters/{meterId}/readings
GET /api/meters/{meterId}/usage?period=monthly
```
