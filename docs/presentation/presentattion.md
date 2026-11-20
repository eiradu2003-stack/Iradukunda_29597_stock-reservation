# Presentation Outline: Warehouse Stock Reservation and Overselling Prevention

**Student:** Iradukunda Emmanuel (ID 29597) | **Course:** INSY 8311 | **Instructor:** Eric Maniraguha

## Slide 1: Warehouse Stock Reservation and Overselling Prevention for Online Orders

- Iradukunda Emmanuel (ID 29597)
- INSY 8311 – Database Development with PL/SQL
- Instructor: Eric Maniraguha
- GitHub: [https://github.com/eiradu2003-stack/Iradukunda_29597_stock-reservation]

## Slide 2: Background and problem

- Online shoppers buy items that the warehouse no longer has.
- The store cancels the order, issues a refund, and the customer loses trust.
- Affected: customers, store staff, and warehouse managers.
- Stock is often checked while browsing, not when the order is confirmed.
- Reservations for unpaid orders are often never released, so stock looks lower than it is.

## Slide 3: Motivation

- Online buying is growing in Rwanda: 30% of households had internet at home in 2023–24 (NISR, 2025).
- Stock control is a database problem: the correct count must be enforced at the moment of ordering.
- I chose this project because it combines real retail data with rules that the database itself enforces.

## Slide 4: Objectives

- Main objective
- Prevent overselling by reserving stock at the moment an order is placed.
- Specific objectives
- Design a 3NF Oracle schema for products, stock, orders, reservations, alerts and auditing.
- Load and validate real online retail transactions.
- Block any order whose quantity exceeds available stock, using a trigger.
- Release expired reservations automatically with DBMS_SCHEDULER.

## Slide 5: Research: existing solutions and gaps

| Source | What it shows | Gap |
| --- | --- | --- |
| Lim et al. (2021), MSOM | Joint optimisation of replenishment, allocation and fulfilment | Optimisation models, not database reservations |
| Arlotto et al. (2026), Operations Research | Regret of inventory placement and online fulfilment | Theoretical; no order-time reservation |
| Shopify Help Center | Zero-stock items blocked unless "continue selling" is on | Allows or blocks a sale; no reservation hold |

- Abstracts read for the two research papers; full texts to be checked before submission.

## Slide 6: Data source: Online Retail II

- UCI Machine Learning Repository (Chen, 2012); CC BY 4.0 licence.
- About 1,067,371 transactions, 1 Dec 2009 to 9 Dec 2011, UK-based non-store retailer.
- Fields: InvoiceNo, StockCode, Description, Quantity, InvoiceDate, UnitPrice, CustomerID, Country.
- Gap in the data: no warehouse stock levels or reservations, so these are generated and labelled as simulations.
- Link: archive.ics.uci.edu/dataset/502/online+retail+ii
- **To complete:** [Insert 3 sample records here after loading the data]

## Slide 7: Proposed solution and innovation

- Oracle PL/SQL database that reserves stock when an order is placed and blocks any order above available stock.
- Innovation option: Smart automation with DBMS_SCHEDULER.
- An hourly job releases expired reservations back to available stock.
- A nightly job writes a row to an ALERTS table for each product below its reorder level.
- Business rules stay in PL/SQL; the scheduler only calls the procedures.

## Slide 8: Planned database

| Table | Purpose |
| --- | --- |
| PRODUCTS | One row per product (stock code, description, price) |
| WAREHOUSE_STOCK | On-hand and reserved quantity per product |
| ORDERS / ORDER_ITEMS | Customer orders and the products on each order |
| RESERVATIONS | Units held for an order, with expiry time |
| RESTOCK_LOG / ALERTS | Stock additions and low-stock warnings |
| AUDIT_LOG | Record of every stock change |

- Relationships: one product has many order items; one order has many reservations.

## Slide 9: PL/SQL approach: where each feature is used

| Feature | Used for |
| --- | --- |
| Function available_qty | Checks free units before an order is confirmed |
| Procedure reserve_stock | Holds every item in an order, or rolls back if any item is short |
| Procedure release_reservation | Frees units when an order is cancelled |
| Trigger trg_block_oversell | Raises error -20001 when an order exceeds available stock |
| Trigger trg_audit_stock | Writes every stock change to AUDIT_LOG |
| Cursor | Loops through reservations past their expiry time |
| Package stock_pkg | Groups reservation, release and availability logic |
| Exceptions | Handles insufficient stock, missing products and invalid quantities |


## Slide 10: Expected results

- A reservation is created when an order is placed, and available stock drops accordingly.
- An order that exceeds available stock is refused with a clear error message.
- An expired, unpaid reservation is released by the scheduled job.
- A low-stock alert is written to ALERTS, and every change appears in AUDIT_LOG.
- **To complete:** [Update each point with the result you have actually demonstrated]

## Slide 11: Future improvements and limitations

- Limitation: stock levels and reservations are simulated, because the source dataset does not record them.
- Limitation: email alerts through UTL_MAIL depend on SMTP access in the Oracle environment.
- Future: connect a Power BI dashboard to show stock and alerts live.
- Future: add a web interface so clerks can place and cancel orders through the stored procedures.

## Slide 12: References

- Arlotto, A., Keskin, I. N., & Wei, Y. (2026). Online demand fulfillment problem with initial inventory placement: A regret analysis. Operations Research. https://doi.org/10.1287/opre.2024.0983
- Chen, D. (2012). Online Retail II [Data set]. UCI Machine Learning Repository. https://doi.org/10.24432/C5CG6D
- Lim, Y. F., Jiu, S., & Ang, M. (2021). Integrating anticipative replenishment allocation with reactive fulfillment for online retailing using robust optimization. Manufacturing & Service Operations Management, 23(6), 1616–1633. https://doi.org/10.1287/msom.2020.0926
- National Institute of Statistics of Rwanda. (2025). Seventh Integrated Household Living Conditions Survey (EICV7): Utilities and amenities thematic report.
- Oracle. (n.d.). DBMS_SCHEDULER. In Oracle Database PL/SQL packages and types reference. Oracle Corporation. https://docs.oracle.com/en/database/
- Shopify. (n.d.). Selling out of stock products. Shopify Help Center.
