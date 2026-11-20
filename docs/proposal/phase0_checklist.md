# Project Proposal

## Warehouse Stock Reservation and Overselling Prevention for Online Orders

**Student:** Iradukunda Emmanuel (ID 29597)
**Course:** INSY 8311
**Instructor:** Eric Maniraguha

---

## 1. Problem Statement

Online retailers sell physical products from limited warehouse stock. When an order is accepted without holding the units it needs, the store can sell more than it has. The customer pays for an item that cannot be delivered, and the business must cancel the order and issue a refund. Demand for online purchasing is growing: 30% of Rwandan households had internet access at home in 2023–24 (National Institute of Statistics of Rwanda, 2025). Existing platforms such as Shopify block sales at zero stock or allow overselling through a setting, but do not describe holding units while an order is pending (Shopify, n.d.). Research has modelled inventory placement and fulfilment mathematically (Arlotto et al., 2026; Lim et al., 2021), but not how a database enforces stock reservations at the moment an order is accepted.

## 2. Proposed Solution

The project builds an Oracle PL/SQL database that reserves stock when an order is placed, blocks any order that exceeds available stock, releases reservations that are cancelled or expire, and logs every stock change. Business rules are enforced in PL/SQL procedures, functions, cursors, triggers and packages, addressing the gap that the optimisation literature leaves at the point of order (Lim et al., 2021). The main objective is to prevent overselling at the moment of ordering. Specific objectives are to design a 3NF schema, load and validate real transaction data, implement the reservation and availability logic, automate expiry with DBMS_SCHEDULER, and record all changes in an audit log.

## 3. Data Source

The Online Retail II dataset (Chen, 2012) from the UCI Machine Learning Repository contains about one million transactions from a UK online retailer, with stock codes, descriptions, quantities, invoice dates, unit prices, customer IDs and countries. It is licensed under CC BY 4.0, which permits educational use with attribution. The dataset does not include warehouse stock levels or reservations, so starting stock and restock records will be generated for each product and documented in the project repository. Customer IDs are numeric codes only and contain no personal details.

## 4. Planned Tables

The schema is designed in third normal form with the following tables: PRODUCTS (product_id, stock_code, description, unit_price); WAREHOUSE_STOCK (stock_id, product_id, on_hand_qty, reserved_qty); CUSTOMERS (customer_id, country); ORDERS (order_id, customer_id, order_date, status); ORDER_ITEMS (item_id, order_id, product_id, quantity); RESERVATIONS (reservation_id, order_id, product_id, qty, reserved_at, expires_at, status); RESTOCK_LOG (restock_id, product_id, qty, restock_date); ALERTS (alert_id, product_id, alert_type, created_at, resolved); and AUDIT_LOG (audit_id, table_name, action, user_name, action_time).

## 5. Planned PL/SQL Features

- **Function `available_qty`:** returns on-hand minus reserved quantity, called before an order is confirmed.
- **Procedure `reserve_stock`:** holds every item in an order, or rolls the whole order back if any product is short.
- **Procedure `release_reservation`:** frees reserved units when an order is cancelled.
- **Trigger `trg_block_oversell`:** raises error -20001 when an order exceeds available stock. This is the main rule that prevents overselling.
- **Trigger `trg_audit_stock`:** writes every WAREHOUSE_STOCK change to AUDIT_LOG.
- **Cursor:** loops through reservations past `expires_at` and releases them.
- **Package `stock_pkg`:** groups the reservation, release and availability logic.
- **Exceptions:** handles insufficient stock, missing products and invalid quantities.

## 6. Innovation

The chosen option is Smart automation, using the DBMS_SCHEDULER package (Oracle, n.d.). An hourly job releases expired reservations and returns their units to available stock. A nightly job writes a row to ALERTS for each product whose available quantity falls below a reorder level. Business rules remain in PL/SQL, and the scheduler only calls the procedures. Email notification through UTL_MAIL is an optional extra, depending on SMTP access in the Oracle environment. The innovation will be demonstrated live in the final presentation.

## References

Arlotto, A., Keskin, I. N., & Wei, Y. (2026). Online demand fulfillment problem with initial inventory placement: A regret analysis. *Operations Research*. https://doi.org/10.1287/opre.2024.0983

Chen, D. (2012). *Online Retail II* [Data set]. UCI Machine Learning Repository. https://doi.org/10.24432/C5CG6D

Lim, Y. F., Jiu, S., & Ang, M. (2021). Integrating anticipative replenishment allocation with reactive fulfillment for online retailing using robust optimization. *Manufacturing & Service Operations Management, 23*(6), 1616–1633. https://doi.org/10.1287/msom.2020.0926

National Institute of Statistics of Rwanda. (2025). *Seventh Integrated Household Living Conditions Survey (EICV7): Utilities and amenities thematic report*. https://www.statistics.gov.rw/sites/default/files/documents/2025-05/EICV7_Thematic%20Report%20Amenities%20and%20Utilities%20%281%29.pdf

Oracle. (n.d.). *DBMS_SCHEDULER*. In *Oracle Database PL/SQL packages and types reference*. Oracle Corporation. https://docs.oracle.com/en/database/

Shopify. (n.d.). *Selling out of stock products*. Shopify Help Center. https://help.shopify.com/en/manual/inventory-and-locations/setup/selling-when-out-of-stock
