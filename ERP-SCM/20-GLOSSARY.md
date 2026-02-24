# ERP-SCM Glossary

## Overview

This glossary defines all domain-specific terms, abbreviations, and technical concepts used throughout the ERP-SCM documentation and codebase.

---

## A

**ABC Analysis**: Inventory classification method that categorizes products into three classes -- A (high value, ~20% of items, ~80% of revenue), B (medium value, ~30% of items, ~15% of revenue), and C (low value, ~50% of items, ~5% of revenue).

**AQL (Acceptable Quality Level)**: The maximum defect rate considered acceptable during sampling inspection. Defined in ANSI/ASQ Z1.4 standard.

**ARIMA (AutoRegressive Integrated Moving Average)**: A time-series forecasting model that combines autoregression, differencing, and moving averages. Used in the demand planning module.

**ASN (Advanced Shipping Notice)**: An electronic notification sent by a supplier to the buyer before a shipment arrives, detailing contents, quantities, and expected delivery.

**ATP (Available-to-Promise)**: The quantity of inventory available to commit to new customer orders, calculated as on-hand stock minus allocated/reserved quantities.

---

## B

**Bias (Forecast)**: A systematic tendency for forecasts to be consistently above (positive bias) or below (negative bias) actual demand. Calculated as the average of (forecast - actual) values.

**Blanket Order**: A long-term purchase agreement with a supplier at pre-negotiated prices, with individual releases scheduled over time.

**BOM (Bill of Materials)**: A hierarchical list of all raw materials, sub-assemblies, and components required to manufacture a finished product, including quantities per unit.

**By-Product**: A secondary product generated during the manufacturing process that was not the primary intended output.

---

## C

**CAPA (Corrective and Preventive Action)**: A formal quality management process for identifying root causes of non-conformances, implementing corrective actions to fix the immediate problem, and preventive actions to avoid recurrence.

**Carrier**: A transportation company (e.g., UPS, FedEx, DHL) responsible for moving goods from origin to destination.

**CloudEvents**: A CNCF specification for describing event data in a common format. Used as the event envelope in ERP-SCM's event-driven architecture.

**CoA (Certificate of Analysis)**: A document issued by a supplier certifying that a product meets specified quality standards, including test results.

**Consensus Planning**: A collaborative demand planning process where statistical forecasts are adjusted with inputs from sales, marketing, and finance to create a unified demand plan.

**CRP (Capacity Requirements Planning)**: The process of determining the amount of machine and labor resources required to fulfill production orders, based on work center capacities and routing times.

**Cross-Docking**: A warehouse practice where incoming goods are immediately transferred to outbound shipping without being stored in the warehouse.

**CV (Coefficient of Variation)**: A measure of demand variability calculated as standard deviation divided by mean. Used in XYZ classification.

**Cycle Count**: A periodic physical inventory counting method where subsets of inventory are counted on a rotating schedule rather than counting all items at once.

---

## D

**Dead Stock**: Inventory items that have had zero demand over a specified period (default: 90 days).

**DOT (Department of Transportation)**: US federal regulatory body governing commercial vehicle operations, driver qualifications, and transportation safety.

**DVLA (Driver and Vehicle Licensing Agency)**: UK government agency responsible for vehicle registration and driver licensing.

---

## E

**EDI (Electronic Data Interchange)**: Standardized electronic format for exchanging business documents (purchase orders, invoices, ASNs) between trading partners.

**ELD (Electronic Logging Device)**: A device that records a driver's hours of service (HOS) for DOT compliance.

**EOQ (Economic Order Quantity)**: The optimal order quantity that minimizes total inventory costs (ordering costs + holding costs). Calculated as: EOQ = sqrt(2DS/H).

**ES (Exponential Smoothing)**: A time-series forecasting method that applies exponentially decreasing weights to past observations. The Holt-Winters variant adds trend and seasonality components.

---

## F

**FIFO (First In, First Out)**: An inventory valuation method where the oldest stock is assumed to be sold first.

**Finite Scheduling**: Production scheduling that respects the actual capacity constraints of work centers, not allowing overloading.

**FOB (Free On Board)**: An Incoterm specifying the point at which the risk of loss transfers from seller to buyer.

---

## G

**GR/IR (Goods Receipt/Invoice Receipt)**: An intermediate clearing account used in the 3-way matching process to reconcile goods received against invoices.

---

## H

**Haversine Formula**: A formula for calculating the great-circle distance between two points on a sphere, used in route optimization for distance calculations.

---

## I

**Incoterms (International Commercial Terms)**: Standardized trade terms (e.g., EXW, FOB, CIF, DDP) that define the responsibilities, costs, and risks associated with the transportation and delivery of goods.

**Infinite Scheduling**: Production scheduling that ignores capacity constraints, useful for identifying required capacity.

**Isolation Forest**: A machine learning algorithm for anomaly detection that isolates observations by randomly selecting features and split values.

---

## K

**Kitting**: The process of assembling individual items into a single kit or bundle for order fulfillment.

---

## L

**Lead Time**: The total time from when an order is placed until the goods are received and available for use.

**LIFO (Last In, First Out)**: An inventory valuation method where the newest stock is assumed to be sold first.

---

## M

**MAD (Mean Absolute Deviation)**: A forecast accuracy metric calculated as the average of absolute differences between forecast and actual values.

**MAPE (Mean Absolute Percentage Error)**: A forecast accuracy metric calculated as the average of absolute percentage errors. MAPE = average(|forecast - actual| / actual * 100%).

**MES (Manufacturing Execution System)**: A shop-floor system that tracks production activities in real-time, recording operator activities, material consumption, and output quantities.

**Min/Max**: An inventory replenishment method where stock is reordered when it falls to the minimum level (reorder point), and ordered up to the maximum level.

**MRP (Material Requirements Planning)**: A production planning system that calculates material requirements by exploding BOMs against demand, netting against current inventory and open orders, and offsetting by lead times.

**mTLS (Mutual TLS)**: A security protocol where both client and server authenticate each other using X.509 certificates.

---

## N

**NCR (Non-Conformance Report)**: A formal document describing a product, process, or material that does not meet specified requirements.

**NN (Nearest Neighbor)**: A heuristic algorithm for route optimization that builds a route by always visiting the closest unvisited stop next.

---

## O

**OIDC (OpenID Connect)**: An identity authentication protocol built on top of OAuth 2.0, used for single sign-on.

**ORM (Object-Relational Mapping)**: A technique for converting between incompatible type systems in object-oriented programming and relational databases. SQLAlchemy is the ORM used in ERP-SCM.

---

## P

**P2P (Procure-to-Pay)**: The end-to-end procurement process from purchase requisition through supplier payment.

**Phantom BOM**: A BOM component that represents a sub-assembly built in-line during production rather than stocked separately.

**POD (Proof of Delivery)**: Documentation (signature, photo) confirming that a shipment was delivered to the recipient.

**Putaway**: The warehouse process of moving received goods from the receiving dock to their designated storage location.

---

## R

**RBAC (Role-Based Access Control)**: An access control method where permissions are assigned to roles, and users are assigned to roles.

**RFQ (Request for Quotation)**: A document sent to suppliers inviting them to submit a price quote for specified goods or services.

**RMA (Return Merchandise Authorization)**: A process for managing the return of goods from customers, including authorization, receiving, inspection, and disposition.

**RLS (Row-Level Security)**: A PostgreSQL feature that enables per-row access control policies on tables, used for multi-tenant data isolation.

---

## S

**S&OP (Sales and Operations Planning)**: A cross-functional business process that aligns demand, supply, and financial plans to achieve strategic objectives.

**Safety Stock**: Extra inventory held as a buffer against demand variability and supply uncertainty. Calculated as: Z * sigma * sqrt(lead_time).

**Slotting Optimization**: The process of determining the optimal storage location for each product in a warehouse to minimize travel time during picking.

**SPC (Statistical Process Control)**: A quality management method using control charts to monitor process stability and detect out-of-control conditions.

**S-Shape Path**: A warehouse picking path strategy where the picker traverses each aisle completely, alternating direction.

---

## T

**TCO (Total Cost of Ownership)**: The total lifecycle cost of an asset, including acquisition, operation, maintenance, fuel, insurance, and disposal costs.

**3-Way Match**: The procurement verification process that compares the purchase order, goods receipt, and supplier invoice to ensure consistency before authorizing payment.

**2-opt**: A local search improvement algorithm for route optimization that iteratively swaps pairs of edges to reduce total distance.

---

## V

**VRP (Vehicle Routing Problem)**: An optimization problem that determines the optimal routes for a fleet of vehicles to serve a set of customers, subject to capacity and time constraints.

---

## W

**WIP (Work-in-Progress)**: Partially completed goods in the manufacturing process, valued at accumulated material, labor, and overhead costs.

**WMS (Warehouse Management System)**: A system for managing warehouse operations including receiving, putaway, picking, packing, and shipping.

---

## X

**XYZ Analysis**: Inventory classification based on demand variability -- X (stable, low variability), Y (moderate variability), Z (high variability).

---

## Z

**Z-Score**: A statistical measure of how many standard deviations a data point is from the mean. Used in anomaly detection with threshold of 2.5 sigma.
