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
| `status` | Enum | `active`, `inactive` |
| `installDate` | Date | Meter installation date |
| `lastReading` | Decimal | Most recent meter reading |
| `lastReadingDate` | Date | Date of most recent reading |
| `unit` | String | Unit of measure (`kWh`) |

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
                                │
                                │ 1:N
                                ▼
┌──────────────────────────────────────────────────────────────────┐
│                           ACCOUNT                                 │
│  - Billing entity                                                │
│  - Receives invoices                                             │
│  - Processes payments                                            │
└──────────────────────────────────────────────────────────────────┘
                │                               │
                │ 1:N                           │ 1:N
                ▼                               ▼
┌─────────────────────────────┐    ┌─────────────────────────────┐
│          PREMISE            │    │           BILL              │
│  - Physical location        │    │  - Invoice for period       │
│  - Service address          │    │  - Payment tracking         │
└─────────────────────────────┘    └─────────────────────────────┘
                │
                │ 1:N
                ▼
┌─────────────────────────────┐
│           METER             │
│  - Electric meters          │
│  - Usage readings           │
└─────────────────────────────┘
```

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
