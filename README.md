[READMETableChief.md](https://github.com/user-attachments/files/27593041/READMETableChief.md)
# 🍽️ TableChief — API Automation

> **End-to-end API test automation suite for the TableChief Restaurant Management System**, built with Postman. Covers all major platform flows across multiple user roles: Super Admin, Admin, Kitchen Staff, Front Desk, and Customer.

---

## 📌 Table of Contents

- [Project Overview](#project-overview)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Base URLs](#base-urls)
- [User Roles & Credentials](#user-roles--credentials)
- [Modules Covered](#modules-covered)
- [Environment Variables](#environment-variables)
- [How to Run](#how-to-run)
  - [Run in Postman](#run-in-postman)
  - [Run via Newman (CLI)](#run-via-newman-cli)
- [CI/CD Integration](#cicd-integration)
- [Test Scripting Approach](#test-scripting-approach)
- [Contributing](#contributing)
- [Author](#author)

---

## Project Overview

**TableChief** is a full-featured Restaurant Management System. This repository contains a comprehensive Postman-based API automation suite that validates the entire platform — from user authentication and restaurant setup to order placement, table reservations, and promotions.

The test suite is designed to be run sequentially, with dynamic chaining between requests using Postman environment variables. Responses from earlier requests (e.g., created IDs, tokens) are automatically stored and reused in downstream tests.

---

## Tech Stack

| Tool | Purpose |
|------|---------|
| **Postman** | API test design, execution & scripting |
| **Newman** | CLI-based collection runner for CI/CD pipelines |
| **JavaScript** | Pre-request & test scripts within Postman |
| **JWT** | Bearer token authentication for all secured endpoints |

---

## Project Structure

```
TableChief-APIAutomation/
├── TableChief.postman_collection.json     # Main test collection (~36K lines)
└── TableCheif.postman_environment.json    # Environment configuration & variables
```

> **Note:** The collection is large (1.56 MB, 36,376 lines) reflecting comprehensive end-to-end coverage across all platform modules.

---

## Base URLs

| Portal | Base URL |
|--------|----------|
| **Admin / Staff Portal** | `https://portal.tablechief.com/api/v1` |
| **Customer Portal** | `https://customer-portal.tablechief.com/api/v1` |

---

## User Roles & Credentials

The suite tests role-based access control across five distinct user types:

| Role | Username (env var) | Notes |
|------|--------------------|-------|
| **Super Admin** | `{{super}}` | Platform-level operations |
| **Admin** | `{{admin}}` | Restaurant-level management |
| **Kitchen Staff** | `{{kitchenstuffusername}}` | Order processing |
| **Front Desk** | `{{frontdeskusername}}` | Reservations & guest management |
| **Customer** | `{{CustomerPhoneNumber}}` | OTP-based login, ordering, reservations |

> ⚠️ Credentials in the environment file are for the test environment only. Do not use or expose production credentials.

---

## Modules Covered

### 🔐 Authentication
- Login for all 5 user roles
- JWT Access Token extraction and chaining to subsequent requests
- OTP-based customer login flow

### 🏪 Restaurant Management
- Create, update, and retrieve restaurant details
- Restaurant slug management
- QR code generation per restaurant

### 📋 Menu Management
- Create and update menus (Dine-in / Takeaway flags)
- Category creation and deletion
- Menu item (food) CRUD operations
- Add-on management per menu item

### 🪑 Table Management
- Create tables with capacity settings
- Update and delete tables
- Table availability tracking during reservations

### 📅 Reservations
- Admin-side reservation creation, update, and status management
- Customer-side reservation booking and cancellation
- Reservation status flow: `Pending` → `Waiting For Fee` → `Active`
- Occupied table tracking

### 🛒 Orders
- Place orders with menu items and add-ons
- Sub-order management
- Order status updates
- Order history by month/year

### 👤 Customer Management
- Customer registration
- Address CRUD (create, update, delete)
- Address slug management
- Guest count tracking

### 🎁 Promotions & Offers
- Create and retrieve promotional offers
- Offer expiry validation
- Expired offer handling

### 🔔 Notifications
- Notification listing and management

### 👔 Role & Staff Management
- Create Admin, Kitchen Staff, and Front Desk roles
- Update staff credentials
- Delete roles

### 🏦 Bank Account Management
- Add and manage bank account details for restaurants

---

## Environment Variables

Key variables managed by the environment file (`TableCheif.postman_environment.json`):

| Variable | Description |
|----------|-------------|
| `url` | Admin portal base URL |
| `CustomerUrl` | Customer portal base URL |
| `AccessToken` | Admin JWT token (auto-updated on login) |
| `AccessTokenKitchenStuff` | Kitchen Staff JWT token |
| `AccessTokenFrontDesk` | Front Desk JWT token |
| `AccessTokenCustomer` | Customer JWT token |
| `newrestaurant_db_id` | ID of the last created restaurant |
| `new_meanu_id` | ID of the last created menu |
| `CatId` | Active category ID |
| `TableID` | Last created table ID |
| `Reservation_id` | Last created admin reservation ID |
| `Customer_Reservation_id` | Last created customer reservation ID |
| `order_id` | Last created order ID |
| `offer_id` | Last created promotion/offer ID |
| `address_id` | Customer address ID |
| `latitudevalue` / `longitudevalue` | Geo-coordinates for address (Dhaka-based) |

> Variables are **automatically populated** by test scripts after each successful API call, enabling seamless request chaining.

---

## How to Run

### Run in Postman

1. **Clone or download** this repository.
2. Open **Postman** and go to **File → Import**.
3. Import both files:
   - `TableChief.postman_collection.json`
   - `TableCheif.postman_environment.json`
4. Select the `0.TableCheif-Final Updated` **environment** from the top-right dropdown.
5. Open the **Collection Runner** (click the ▶ button next to the collection).
6. Select all folders or a specific module folder.
7. Click **Run TableChief** to execute.

---

### Run via Newman (CLI)

Newman allows running the collection from the command line, ideal for CI/CD pipelines.

**Prerequisites:**
```bash
npm install -g newman
npm install -g newman-reporter-htmlextra  # For HTML reports
```

**Run the full collection:**
```bash
newman run TableChief.postman_collection.json \
  -e TableCheif.postman_environment.json \
  --reporters cli,htmlextra \
  --reporter-htmlextra-export reports/TableChief-TestReport.html
```

**Run a specific folder:**
```bash
newman run TableChief.postman_collection.json \
  -e TableCheif.postman_environment.json \
  --folder "Menu Management" \
  --reporters cli
```

---

## CI/CD Integration

To integrate with **GitHub Actions**, add the following workflow file at `.github/workflows/api-tests.yml`:

```yaml
name: TableChief API Tests

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  api-test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      - name: Install Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'

      - name: Install Newman
        run: npm install -g newman newman-reporter-htmlextra

      - name: Run API Tests
        run: |
          newman run TableChief.postman_collection.json \
            -e TableCheif.postman_environment.json \
            --reporters cli,htmlextra \
            --reporter-htmlextra-export reports/TestReport.html

      - name: Upload Test Report
        uses: actions/upload-artifact@v3
        if: always()
        with:
          name: API-Test-Report
          path: reports/TestReport.html
```

---

## Test Scripting Approach

Each request leverages Postman's built-in scripting capabilities:

**Pre-request Scripts** — used to:
- Generate dynamic/random test data (names, emails, phone numbers)
- Set computed variable values before sending requests

**Test Scripts** — used to:
- Assert HTTP status codes (e.g., `pm.response.to.have.status(200)`)
- Validate response schema and field values
- Extract and store response data into environment variables for chained requests
- Verify business logic (e.g., reservation status transitions)

**Example test script pattern:**
```javascript
// Assert status
pm.test("Status code is 201", () => {
    pm.response.to.have.status(201);
});

// Chain ID to next request
const response = pm.response.json();
pm.environment.set("TableID", response.id);

// Validate field
pm.test("Table capacity matches", () => {
    pm.expect(response.capacity).to.eql(pm.environment.get("TableCapacity"));
});
```

---

## Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/add-payment-module`
3. Add or update tests in the Postman collection
4. Export the updated collection and environment files
5. Submit a Pull Request with a clear description of what was added/changed

---

## Author

**Imran Hassan**
QA Engineer | API & UI Automation
[GitHub Profile](https://github.com/imranhassanaiub)

---

> 📬 For issues or suggestions, feel free to open a [GitHub Issue](https://github.com/imranhassanaiub/TableChief-APIAutomation/issues).
