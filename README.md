# Warehouse Stock Reservation and Overselling Prevention for Online Orders

**Name:** Iradukunda Emmanuel
**Student ID:** 29597
**Course:** INSY 8311 – Database Development with PL/SQL
**Instructor:** Eric Maniraguha

---

## Problem Statement

Online retailers sell physical products from a limited warehouse stock. When an order is accepted without holding the units it needs, the store can sell more items than it has. The customer then pays for something that cannot be delivered, and the business must cancel the order, refund the money and risk losing the customer's trust. Stock is often checked when a customer browses the site rather than when the order is confirmed, and reservations that customers never pay for are often never released, so available stock is understated for other buyers.

## Objectives

**Main objective:** Prevent overselling by reserving warehouse stock at the moment an order is placed, and block any order that exceeds the available quantity.

**Specific objectives:**
- Design a normalised (3NF) Oracle schema for products, stock, orders, reservations, restocks, alerts and auditing.
- Load real online retail transactions into the schema and validate the loaded data.
- Implement a function that calculates available quantity (on-hand minus reserved).
- Implement procedures that reserve stock for an order and release it when an order is cancelled.
- Enforce the no-overselling rule with a database trigger and a custom error message.
- Automate the release of expired reservations and the creation of low-stock alerts using DBMS_SCHEDULER.
- Record every stock change in an audit log.

## Data Source

- **Dataset:** Online Retail II, UCI Machine Learning Repository (Chen, 2012).
- **Link:** https://archive.ics.uci.edu/dataset/502/online+retail+ii
- **Licence:** Creative Commons Attribution 4.0 (CC BY 4.0), which permits educational use with attribution.
- **Size and fields:** About one million transaction rows from a UK-based online retailer, with invoice number, stock code, description, quantity, invoice date, unit price, customer ID and country.
- **Limitation:** The dataset does not contain warehouse stock levels or reservations. Starting stock quantities and restock records will be generated for each product, and this will be documented in the data folder and in the presentation.
- **Personal data:** Customer IDs are numeric codes only. No names or contact details are used.

## Innovation

**Option chosen:** Smart automation (DBMS_SCHEDULER).

The database runs on its own schedule. An hourly job releases reservations whose expiry time has passed, returning their units to available stock. A second job scans for products whose available quantity falls below a reorder level and writes a row to the ALERTS table for each one. Business rules remain in PL/SQL: the scheduler only calls the procedures. Email notification through UTL_MAIL is a planned extra and will only be used if the SMTP settings are available in the Oracle environment.

- **Demonstration:** Live run of the scheduler job, followed by the change in RESERVATIONS and ALERTS tables in SQL Developer.
- **Documentation:** The innovation/ folder will contain a description, the run instructions, and screenshots.

## Planned Tables

| Table | Key columns |
|---|---|
| PRODUCTS | product_id, stock_code, description, unit_price |
| WAREHOUSE_STOCK | stock_id, product_id, on_hand_qty, reserved_qty |
| CUSTOMERS | customer_id, country |
| ORDERS | order_id, customer_id, order_date, status |
| ORDER_ITEMS | item_id, order_id, product_id, quantity |
| RESERVATIONS | reservation_id, order_id, product_id, qty, reserved_at, expires_at, status |
| RESTOCK_LOG | restock_id, product_id, qty, restock_date |
| ALERTS | alert_id, product_id, alert_type, created_at, resolved |
| AUDIT_LOG | audit_id, table_name, action, user_name, action_time |

## Planned PL/SQL Features

- **Function `available_qty(product_id)`:** returns on-hand quantity minus reserved quantity. The order screen calls it before an order is confirmed.
- **Procedure `reserve_stock(order_id)`:** creates reservations for every item in an order. Rolls back the whole order if any product is short.
- **Procedure `release_reservation(order_id)`:** frees the reserved units when an order is cancelled.
- **Trigger `trg_block_oversell`:** raises a custom error (-20001) on ORDER_ITEMS when the quantity exceeds available stock. This is the main rule that prevents overselling.
- **Trigger `trg_audit_stock`:** writes every change to WAREHOUSE_STOCK into AUDIT_LOG.
- **Cursor:** loops through reservations past expires_at and releases them.
- **Package `stock_pkg`:** groups the reservation, release and availability logic.
- **Exception handling:** handles insufficient stock (custom error -20001), missing products and invalid quantities.
- **DBMS_SCHEDULER jobs:** hourly expiry clean-up and nightly low-stock check.

## References

Chen, D. (2012). *Online Retail II* [Data set]. UCI Machine Learning Repository. https://doi.org/10.24432/C5CG6D

Oracle. (n.d.). *Oracle Database PL/SQL language reference*. Oracle Corporation. https://docs.oracle.com/en/database/

Oracle. (n.d.). *DBMS_SCHEDULER*. In *Oracle Database PL/SQL packages and types reference*. Oracle Corporation. https://docs.oracle.com/en/database/

[Research paper 1 in APA format, added after reading]

[Research paper 2 in APA format, added after reading]

[Industry report or statistic, e.g. from NISR Rwanda or World Bank, added after reading]
