# thrift-store — User Stories

Online store for new and used clothing. Payment is **Pix only**, processed by **Stripe**.

**Priority:** M = Must have (MVP) · S = Should have · C = Could have  
**Estimate:** execution hours (backend + frontend + tests). Initial estimates, to be calibrated with the actual time logged in Jira during the first sprints.

## Personas

- **Customer:** buys items on mobile or desktop.
- **Administrator:** the store owner; manages items and orders.

## Business rules

| ID | Rule |
|---|---|
| BR01 | A **used** item is unique (quantity = 1). Only one customer can buy it. |
| BR02 | A **new** item has stock per variation (size/color). |
| BR03 | When a Pix charge is created, the item is **reserved** for 30 minutes. If unpaid, the reservation expires and the item returns to the storefront. |
| BR04 | If two customers try to reserve the same item at the same time, only one succeeds. |
| BR05 | The order stores the **price at purchase time**. Changing the item price later does not affect existing orders. |
| BR06 | Items are never deleted from the database: they are **archived** (soft delete) to preserve order history and reports. |
| BR07 | A sold or reserved item does not appear as available in the storefront. |
| BR08 | Order status flow: `AWAITING_PAYMENT` -> `PAID` -> `SHIPPED` -> `DELIVERED`; or `CANCELLED` / `EXPIRED`. |
| BR09 | Passwords are stored hashed. The system never stores card data (payments are Pix only, processed by Stripe). |

## Epic 1 — Foundation

**US-01 · Set up development environment and CI pipeline** · M · 20h
As a **developer**, I want a project with database, migrations, Docker and CI configured, so that I can build on a reliable foundation.
- `docker compose up` starts API, database and frontend
- Migrations run automatically on startup
- Every push runs build and tests on GitHub Actions

## Epic 2 — Authentication

**US-02 · Customer sign-up** · M · 12h
As a **customer**, I want to create an account, so that I can place orders.
- Requires name, email and password
- Duplicate email is rejected with a clear message
- Password has at least 8 characters
- Sign-up asks for consent to data usage (LGPD)

**US-03 · Login and access control** · M · 14h
As a **customer or administrator**, I want to sign in with email and password, so that I can access my features.
- Valid credentials return an access token
- Invalid credentials return an error without revealing which field was wrong
- Admin routes reject customers with a permission error

**US-04 · Password recovery** · S · 10h
As a **customer**, I want to reset my password by email, so that I can regain access to my account.
- Reset link expires in 1 hour
- Reset link can be used only once

## Epic 3 — Catalog

**US-05 · Create item** · M · 16h
As a **administrator**, I want to create a new or used item, so that it becomes available in the store.
- Fields: name, description, category, price, type (new/used)
- Used items require condition (e.g. excellent, good, with flaws), size and measurements
- New items allow size/color variations with stock
- Price must be greater than zero

**US-06 · Item photos** · M · 10h
As a **administrator**, I want to upload several photos per item, so that customers can see the real piece.
- Accepts JPG and PNG up to a defined size limit
- One photo is marked as the main photo
- Used items require at least one photo

**US-07 · Edit item** · M · 6h
As a **administrator**, I want to edit item details, so that I can fix information and prices.
- Changes appear immediately in the storefront
- Previous orders keep the original price (BR05)

**US-08 · Archive item** · M · 4h
As a **administrator**, I want to remove an item from the storefront, so that I can take down items I no longer sell.
- An archived item disappears from the storefront but stays in past orders (BR06)
- An item with an active reservation cannot be archived

**US-09 · Manage categories** · S · 6h
As a **administrator**, I want to create and edit categories, so that I can organize the store.
- A category with active items cannot be removed

**US-10 · Browse available items** · M · 10h
As a **customer**, I want to see the list of available items, so that I can choose what to buy.
- Paginated list
- Shows main photo, name, price and whether it is new or used
- Sold or reserved items do not appear (BR07)

**US-11 · Search and filter items** · M · 10h
As a **customer**, I want to filter by category, size, price and type, so that I can quickly find something that fits me.
- Filters can be combined
- When nothing matches, a clear message is shown
- Text search on the item name

**US-12 · Item detail page** · M · 8h
As a **customer**, I want to see the full details of an item, so that I can decide whether to buy it.
- Photo gallery, description, size, measurements and condition
- An unavailable item shows a notice and no buy button

## Epic 4 — Cart

**US-13 · Shopping cart** · M · 12h
As a **customer**, I want to add and remove items from the cart, so that I can gather my purchase.
- A used item can be added only once
- The total is recalculated on every change
- If an item becomes unavailable, the cart warns and removes it

## Epic 5 — Checkout and Pix Payment

**US-14 · Delivery address** · M · 8h
As a **customer**, I want to register my address, so that I can receive my order.
- Required fields are validated (including postal code / CEP)
- A customer can have more than one address and pick one at checkout

**US-15 · Place order and reserve items** · M · 16h
As a **customer**, I want to check out my cart, so that my items are secured.
- Creates an `AWAITING_PAYMENT` order with frozen prices (BR05)
- Items are reserved for 30 minutes (BR03)
- If another customer already reserved an item, the order is not created and the customer is notified (BR04)

**US-16 · Pay with Pix (Stripe)** · M · 12h
As a **customer**, I want to pay with Pix, so that I can complete my purchase.
- Customer is redirected to a Stripe-hosted page with the Pix QR code and copy-and-paste code
- Pix expiration matches the reservation time (BR03)
- Amounts are in BRL

**US-17 · Payment confirmation via webhook** · M · 12h
As a **customer**, I want my payment to be confirmed automatically, so that I do not need to notify the store.
- On Stripe confirmation, the order becomes `PAID`
- A repeated notification does not process the payment twice
- A notification with an invalid signature is rejected

**US-18 · Reservation expiration** · M · 8h
As a **system**, I want to release items from unpaid orders, so that they return to the storefront.
- An order unpaid after 30 minutes becomes `EXPIRED`
- Reserved items become available again

**US-19 · Track my orders** · M · 8h
As a **customer**, I want to see my orders and their status, so that I can follow my purchase.
- List with date, total and status
- Detail shows items, address and tracking code, if any

## Epic 6 — Administration

**US-20 · Manage orders** · S · 10h
As a **administrator**, I want to view orders and update their status, so that I can handle shipping.
- Marking as `SHIPPED` requires a tracking code
- Invalid status transitions are blocked (BR08)
- Orders can be filtered by status

**US-21 · Monthly sales report** · S · 12h
As a **administrator**, I want a monthly report, so that I can track store performance.
- Month and year selection
- Shows total revenue, number of items sold, new vs used split and sales by category
- Considers only paid orders or later

**US-22 · Shipping cost calculation** · S · 10h
As a **customer**, I want to see the shipping cost before paying, so that I know the total cost.
- Shipping calculated by postal code range (initial version)
- The value is included in the order total

## Epic 7 — Quality and Delivery

**US-23 · Documentation and presentation** · M · 20h
As a **recruiter or developer**, I want to understand the project quickly, so that I can evaluate the quality of the work.
- README with problem, stack, how to run and screenshots
- Diagrams (C4, ERD, checkout sequence, order states) in the `docs` folder
- ADRs for the main decisions
- API documented with OpenAPI/Swagger

**Estimated total: ~254h.** Tests accompany each story. Card payments, favorites, email notifications and a multi-seller marketplace are out of scope for now.
