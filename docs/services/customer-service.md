# Customer Service

The Customer Service manages customer lifecycle, accounts, premises, and meters. It is the central service for customer-related operations in OSSREP.

## Overview

**Repository:** `customer-service`

**Key Responsibilities:**

- Customer enrollment and management
- Account lifecycle operations
- Premise management
- Meter registration and tracking
- Customer preferences and communication settings

## Domain Model

See [Domain Model](../architecture/domain-model.md) for detailed entity descriptions and relationships.

### Entity Hierarchy

```
Customer → Accounts → Premises → Meters
```

## Enrollment Flow

The Customer Portal provides a 7-step guided enrollment flow for new customers:

### Step 1: Customer Type

Select the type of customer account:

- **Individual / Residential** - For homeowners and renters
- **Business / Commercial** - For businesses, organizations, and commercial properties

### Step 2: Contact Information

Collect customer contact details:

**Individual customers:**

- First name, last name
- Email address
- Phone number

**Business customers:**

- Legal business name
- Business type (LLC, Corporation, etc.)
- Tax ID / EIN
- Business email and phone
- Primary contact person details

### Step 3: Service Address & Start Date

The physical premise address where electricity service is needed:

- Street address
- Apartment/suite/unit (optional)
- City, state, ZIP code
- **Requested service start date** - When the customer wants service to begin

The start date is used to create the contract with calculated end date based on the selected plan's term.

### Step 4: Plan Selection

After the service address is entered, available plans for that location are displayed:

- Plans are filtered by service area/ZIP code
- Each plan displays rate per kWh, term length, and features
- Plan types include fixed-rate, variable, and indexed options
- Renewable energy options (wind, solar) with percentage shown
- Early termination fees displayed for contract plans
- Links to required disclosure documents:
    - **EFL** (Electricity Facts Label) - standardized pricing disclosure
    - **TOS** (Terms of Service) - contract terms and conditions
    - **YRAC** (Your Rights as a Customer) - consumer rights information

The selected plan determines the customer's energy rate and contract terms.

### Step 5: Meter Selection

After selecting a plan, available meters at the premise are retrieved:

- System looks up meters registered at the address
- Individual customers typically see a single meter
- Business customers may see multiple meters (e.g., different buildings, units, or circuits)
- Customer selects one or more meters to enroll for service
- At least one meter must be selected to proceed

This step ensures customers only enroll for meters at their specific premise location.

### Step 6: Mailing Address

Billing/mailing address for correspondence:

- Option to use service address as mailing address
- Or specify a separate mailing address

### Step 7: Review & Terms

Final enrollment summary and terms acceptance:

- Bill delivery preference (email, mail, or both)
- Marketing opt-in/out
- Terms of service acceptance (including EFL, TOS, YRAC acknowledgment)
- Summary showing:
    - Customer info and contact details
    - Service address and selected meters
    - Selected plan with rate and features
    - **Contract details**: start date, end date, term length, and early termination fee (if applicable)

### Confirmation Page

Upon successful enrollment submission, customers are redirected to a confirmation page:

- **Confirmation Number** - Unique reference for the enrollment (e.g., `ENR-M5X2Y3-AB4C`)
- **Complete Summary** - All enrollment details displayed in organized cards:
    - Customer information
    - Service address and enrolled meters
    - Plan details (rate, term, features)
    - Contract dates (start and end)
- **What Happens Next** - Timeline showing:
    1. Confirmation email will be sent
    2. Service activation on the start date
    3. First bill after initial billing cycle
- **Actions** - Print summary button and link to sign in

A Contract is created linking the selected plan to the customer's accounts.

## Customer Types

The service supports two customer types with different organizational structures:

### Individual Customers

Residential consumers with simpler account structures:

- Typically one account
- One or more premises (primary residence, vacation homes, rentals)
- One or more meters per premise

### Business Customers

Commercial/industrial customers with complex organizational needs:

- Multiple accounts (by department, cost center, location group)
- Multiple premises per account
- Multiple meters per premise
- Aggregated billing and reporting across all accounts

## API Endpoints

### Enrollment Operations

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/enrollment` | Submit new customer enrollment |
| `GET` | `/api/enrollment/plans` | Get available plans for a ZIP code |
| `GET` | `/api/enrollment/plans/{planId}` | Get plan details with EFL, TOS, YRAC |
| `GET` | `/api/enrollment/meters` | Look up available meters at an address |
| `POST` | `/api/enrollment/validate-address` | Validate service address |

### Customer Operations

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/customer` | Get current customer profile |
| `PUT` | `/api/customer` | Update customer profile |
| `GET` | `/api/customer/summary` | Get aggregated customer summary |

### Account Operations

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/accounts` | List all accounts for customer |
| `GET` | `/api/accounts/{accountId}` | Get account details |
| `PUT` | `/api/accounts/{accountId}` | Update account (nickname, etc.) |
| `GET` | `/api/accounts/{accountId}/summary` | Get account summary with balances |

### Premise Operations

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/accounts/{accountId}/premises` | List premises for account |
| `GET` | `/api/premises/{premiseId}` | Get premise details |
| `PUT` | `/api/premises/{premiseId}` | Update premise |
| `POST` | `/api/premises/{premiseId}/set-primary` | Set as primary premise |

### Meter Operations

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/premises/{premiseId}/meters` | List meters at premise |
| `GET` | `/api/meters/{meterId}` | Get meter details |
| `GET` | `/api/meters/{meterId}/readings` | Get meter reading history |

### Preferences Operations

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/preferences` | Get customer preferences |
| `PUT` | `/api/preferences` | Update preferences |
| `GET` | `/api/preferences/notifications` | Get notification settings |
| `PUT` | `/api/preferences/notifications` | Update notification settings |

### Contract Operations

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/contracts` | List all contracts for customer |
| `GET` | `/api/contracts/{contractId}` | Get contract details |
| `GET` | `/api/contracts/active` | Get currently active contract(s) |
| `POST` | `/api/contracts/{contractId}/renew` | Renew an expiring contract |
| `POST` | `/api/contracts/{contractId}/cancel` | Cancel contract (ETF may apply) |
| `GET` | `/api/contracts/{contractId}/accounts` | List accounts covered by contract |

## Data Models

### Available Plans Response

```json
{
  "zipCode": "75001",
  "plans": [
    {
      "planId": 1001,
      "name": "Green Energy 12",
      "description": "Lock in a competitive rate with 100% renewable energy from Texas wind farms.",
      "ratePerKwh": 11.9,
      "termMonths": 12,
      "earlyTerminationFee": 150.00,
      "baseCharge": 9.95,
      "renewablePercent": 100,
      "planType": "fixed",
      "features": ["100% Wind Energy", "Fixed Rate", "No Hidden Fees"],
      "documents": {
        "eflUrl": "/api/enrollment/plans/1001/efl",
        "tosUrl": "/api/enrollment/plans/1001/tos",
        "yracUrl": "/api/enrollment/plans/1001/yrac"
      }
    },
    {
      "planId": 1002,
      "name": "Value Saver 24",
      "description": "Our best long-term value with a low fixed rate locked in for 2 years.",
      "ratePerKwh": 10.5,
      "termMonths": 24,
      "earlyTerminationFee": 200.00,
      "baseCharge": 4.95,
      "renewablePercent": 15,
      "planType": "fixed",
      "features": ["Lowest Rate", "24-Month Price Lock", "Low Base Charge"],
      "documents": {
        "eflUrl": "/api/enrollment/plans/1002/efl",
        "tosUrl": "/api/enrollment/plans/1002/tos",
        "yracUrl": "/api/enrollment/plans/1002/yrac"
      }
    }
  ]
}
```

### Enrollment Request

```json
{
  "customerType": "individual",
  "firstName": "John",
  "lastName": "Smith",
  "email": "john.smith@example.com",
  "phone": "(555) 123-4567",
  "serviceAddress": {
    "street": "123 Main Street",
    "unit": null,
    "city": "Anytown",
    "state": "TX",
    "zip": "75001"
  },
  "serviceStartDate": "2025-02-01",
  "selectedPlanId": 1001,
  "selectedMeterIds": [3001],
  "mailingAddress": {
    "sameAsService": true
  },
  "preferences": {
    "billDelivery": "email",
    "marketingOptIn": false
  },
  "acceptedTerms": true
}
```

### Enrollment Response

```json
{
  "success": true,
  "confirmationNumber": "ENR-M5X2Y3-AB4C",
  "customerId": 1,
  "contractId": 5001,
  "contract": {
    "contractNumber": "CTR-2025-00001",
    "startDate": "2025-02-01",
    "endDate": "2026-02-01",
    "status": "pending"
  },
  "accountId": 1001,
  "accountNumber": "1234567890",
  "enrolledMeters": [
    {
      "meterId": 3001,
      "meterNumber": "E-12345678"
    }
  ],
  "submittedAt": "2025-01-15T14:30:00Z",
  "nextSteps": [
    "Confirmation email sent to john.smith@example.com",
    "Service activation scheduled for Feb 1, 2025",
    "First bill will be generated after initial billing cycle"
  ]
}
```

### Business Enrollment Request

```json
{
  "customerType": "business",
  "businessName": "Acme Corporation",
  "businessType": "corporation",
  "taxId": "12-3456789",
  "email": "accounts@acme.com",
  "phone": "(555) 987-6543",
  "primaryContact": {
    "firstName": "Jane",
    "lastName": "Doe",
    "title": "Office Manager",
    "email": "jane.doe@acme.com",
    "phone": "(555) 987-6544"
  },
  "serviceAddress": {
    "street": "100 Corporate Drive",
    "unit": "Suite 500",
    "city": "Dallas",
    "state": "TX",
    "zip": "75201"
  },
  "serviceStartDate": "2025-02-15",
  "selectedPlanId": 1002,
  "selectedMeterIds": [3001, 3002, 3003],
  "mailingAddress": {
    "sameAsService": false,
    "street": "PO Box 1234",
    "city": "Dallas",
    "state": "TX",
    "zip": "75221"
  },
  "preferences": {
    "billDelivery": "both",
    "marketingOptIn": true
  },
  "acceptedTerms": true
}
```

### Contract Response

```json
{
  "contractId": 5001,
  "contractNumber": "CTR-2025-00001",
  "customerId": 1,
  "status": "active",
  "startDate": "2025-02-01",
  "endDate": "2026-02-01",
  "signedAt": "2025-01-15T14:30:00Z",
  "plan": {
    "planId": 1001,
    "name": "Green Energy 12",
    "ratePerKwh": 11.9,
    "termMonths": 12,
    "planType": "fixed"
  },
  "earlyTerminationFee": 150.00,
  "accounts": [
    {
      "accountId": 1001,
      "accountNumber": "1234567890",
      "addedAt": "2025-01-15T14:30:00Z"
    }
  ]
}
```

### Contract List Response

```json
{
  "contracts": [
    {
      "contractId": 5001,
      "contractNumber": "CTR-2025-00001",
      "status": "active",
      "startDate": "2025-02-01",
      "endDate": "2026-02-01",
      "planName": "Green Energy 12",
      "accountCount": 1
    },
    {
      "contractId": 4001,
      "contractNumber": "CTR-2024-00015",
      "status": "expired",
      "startDate": "2024-02-01",
      "endDate": "2025-02-01",
      "planName": "Value Saver 12",
      "accountCount": 1
    }
  ]
}
```

### Available Meters Response

```json
{
  "address": {
    "street": "123 Main Street",
    "city": "Anytown",
    "state": "TX",
    "zip": "75001"
  },
  "meters": [
    {
      "meterId": 3001,
      "meterNumber": "E-12345678",
      "type": "electric",
      "status": "available"
    },
    {
      "meterId": 3002,
      "meterNumber": "E-23456789",
      "type": "electric",
      "status": "available"
    }
  ]
}
```

### Customer Response

```json
{
  "customerId": 1,
  "type": "individual",
  "firstName": "John",
  "lastName": "Smith",
  "email": "john.smith@example.com",
  "phone": "(555) 123-4567",
  "status": "active",
  "createdAt": "2020-03-15T00:00:00Z",
  "accounts": [
    {
      "accountId": 1001,
      "accountNumber": "1234567890",
      "status": "active"
    }
  ]
}
```

### Business Customer Response

```json
{
  "customerId": 2,
  "type": "business",
  "businessName": "Acme Corporation",
  "email": "accounts@acme.com",
  "phone": "(555) 987-6543",
  "status": "active",
  "createdAt": "2019-01-15T00:00:00Z",
  "accounts": [
    {
      "accountId": 1002,
      "accountNumber": "9876543210",
      "nickname": "Headquarters",
      "status": "active"
    },
    {
      "accountId": 1003,
      "accountNumber": "5555555555",
      "nickname": "Warehouse District",
      "status": "active"
    }
  ]
}
```

### Account Detail Response

```json
{
  "accountId": 1001,
  "accountNumber": "1234567890",
  "nickname": null,
  "status": "active",
  "currentBalance": 142.50,
  "lastPaymentDate": "2024-12-15",
  "lastPaymentAmount": 128.75,
  "premises": [
    {
      "premiseId": 2001,
      "street": "123 Main Street",
      "unit": null,
      "city": "Anytown",
      "state": "TX",
      "zip": "75001",
      "isPrimary": true,
      "meters": [
        {
          "meterId": 3001,
          "meterNumber": "E-12345678",
          "type": "electric",
          "status": "active"
        }
      ]
    }
  ]
}
```

### Customer Summary Response

```json
{
  "totalAccounts": 3,
  "totalPremises": 5,
  "totalMeters": 6,
  "totalCurrentBalance": 5327.55,
  "totalDueAmount": 5327.55,
  "nextDueDate": "2025-01-22",
  "usageByType": {
    "electric": {
      "usage": 109280,
      "unit": "kWh"
    }
  }
}
```

### Meter Detail Response

```json
{
  "meterId": 3001,
  "meterNumber": "E-12345678",
  "type": "electric",
  "status": "active",
  "installDate": "2020-03-15",
  "lastReading": 1245,
  "lastReadingDate": "2025-01-10",
  "unit": "kWh",
  "premise": {
    "premiseId": 2001,
    "street": "123 Main Street",
    "city": "Anytown",
    "state": "TX",
    "zip": "75001"
  }
}
```

### Preferences Response

```json
{
  "billDelivery": "email",
  "notifications": {
    "billReady": true,
    "paymentDue": true,
    "outageAlerts": true,
    "highUsageAlerts": false
  },
  "autoPay": {
    "enabled": false,
    "paymentMethodId": null
  },
  "mailingAddress": {
    "sameAsPrimaryPremise": true,
    "street": null,
    "city": null,
    "state": null,
    "zip": null
  }
}
```

## Events

The Customer Service publishes events to Kafka for cross-service communication:

| Event | Description | Consumers |
|-------|-------------|-----------|
| `EnrollmentSubmitted` | New enrollment request received | Workflow |
| `EnrollmentCompleted` | Enrollment processing completed | Notifications |
| `CustomerCreated` | New customer enrolled | Billing, Market |
| `CustomerUpdated` | Customer profile changed | Billing |
| `AccountCreated` | New account added | Billing |
| `AccountStatusChanged` | Account activated/deactivated | Billing, Market |
| `ContractCreated` | New contract created during enrollment | Billing, Market |
| `ContractActivated` | Contract start date reached | Billing |
| `ContractExpiring` | Contract approaching end date | Notifications |
| `ContractExpired` | Contract term ended | Billing |
| `ContractRenewed` | Customer renewed into new contract | Billing, Market |
| `ContractCancelled` | Contract cancelled early | Billing |
| `PremiseAdded` | New premise added | Meter, Market |
| `MeterEnrolled` | Meter selected during enrollment | Meter, Market |
| `MeterInstalled` | Meter added to premise | Meter, Billing |
| `MeterRemoved` | Meter removed from premise | Meter, Billing |
| `PreferencesUpdated` | Customer preferences changed | Billing, Notifications |

## Integration Points

### Billing Service

- Receives account and customer data for invoice generation
- Notified of account status changes
- Queries customer preferences for bill delivery

### Meter Service

- Receives meter registration data
- Reports meter readings back to Customer Service
- Coordinates meter exchange operations

### Market Service

- Receives customer/premise data for market enrollments
- Coordinates move-in/move-out transactions
- Reports market-initiated changes

## Customer Portal Features

The Customer Portal (`ossrep-customer-portal`) provides self-service access to Customer Service capabilities:

### Enrollment (Start Service)

7-step guided enrollment for new customers with confirmation page:

1. **Customer Type** - Individual or business selection
2. **Contact Information** - Customer and contact details
3. **Service Address & Start Date** - Premise location and requested start date
4. **Plan Selection** - Choose electricity plan (rate, term, features)
5. **Meter Selection** - Select meters at the premise to enroll
6. **Mailing Address** - Billing address configuration
7. **Review & Terms** - Summary and terms acceptance
8. **Confirmation Page** - Complete summary with confirmation number and next steps

The plan selection step displays available electricity plans with rates, contract terms, renewable energy options, and links to required disclosure documents (EFL, TOS, YRAC). Upon completion, customers receive a confirmation number and can print their enrollment summary.

### Dashboard

- Aggregated view across all accounts (especially for business customers)
- Total balance due with next due date
- Account/premise/meter counts
- Usage summary by meter type

### Account Management

- View all accounts with balances
- Premises with meter details
- Meter reading history

### Profile & Preferences

- Update contact information
- Manage mailing address
- Set communication preferences
- Configure notification settings
- Enable/disable auto-pay

## Security

### Authentication

- OIDC/JWT authentication via Keycloak
- Customer ID derived from JWT `sub` claim

### Authorization

- Customers can only access their own data
- Account ownership verified on all operations
- Admin endpoints require elevated roles

### Data Privacy

- PII encrypted at rest
- Audit logging for all data access
- GDPR/CCPA compliance considerations
