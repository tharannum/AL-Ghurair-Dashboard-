 # Al Ghurair - Procurement & Inventory Performance Analytics

## Executive Summary

**Al Ghurair**, a leading industrial and commercial conglomerate in the UAE, manages large-scale purchase orders, multi-facility warehouse inventories, and vendor networks across Dubai, Abu Dhabi, Sharjah, Ras Al Khaimah, and KIZAD. However, disconnected financial spend tracking and physical warehouse operations create operational bottlenecks, untracked vendor lead-time delays, and risk of stockouts.

This Business Intelligence project analyzes Al Ghurair’s procurement and warehouse dataset using Power BI to unify financial commitments with inventory operations. The goal is to provide clear, executive-level visibility across procurement spend, supplier SLA reliability, physical inventory valuation, and warehouse material flows.

![Al Ghurair](https://github.com/tharannum/AL-Ghurair-Dashboard-/blob/main/Al%20ghurair.png)

The analysis includes:

* **Procurement Spend Analysis:** Total spend commitments, order volume, and category distribution over time.
* **Supplier SLA & Performance:** On-Time Delivery (OTD) rates, contractual lead-time variances, and vendor dependency risks.
* **Inventory Valuation & Stockout Risks:** Total locked-up capital per warehouse and dynamic alerts for SKUs below safety thresholds.
* **Warehouse Operations & Stock Flows:** Inbound vs. outbound material throughput ratios across UAE facilities.

An interactive 2-page Power BI dashboard suite has been built to support data-driven decisions for procurement, logistics, supply chain planners, and executive leadership.

---

## 🗂️ 1. Project Overview

The purpose of this Power BI project is to convert raw procurement and warehouse log data into a structured dashboard that enables Al Ghurair to answer critical questions:

* Where capital is committed versus when goods physically arrive.
* Which vendors breach delivery SLAs and cause operational lead-time delays.
* Which product categories and suppliers account for the largest cost concentration.
* How much capital is locked up in stock across different UAE warehouse facilities.
* Which critical SKUs have breached reorder thresholds and require urgent procurement.
* How efficiently stock flows in and out of different regional logistics hubs.

---

## 🗃️ 2. Data Description

### Dataset Source

Cleaned transaction logs exported from Al Ghurair’s Enterprise Resource Planning (ERP) and Warehouse Management System (WMS).

### Included Fields

* **Purchase Orders:** PO Number, Order Date, Promised Delivery Date, Actual Delivery Date, PO Value (AED), Order Status.
* **Supplier Information:** Supplier ID, Supplier Name, Contract Lead Time (Days), Delivery Performance Logs.
* **Inventory Details:** Product ID, Product Name, Category, Stock On Hand (Units), Reorder Level (Units), Unit Cost (AED).
* **Warehouse & Logistics:** Warehouse ID, Warehouse Name, Location (City/Zone), Movement Type (`IN` / `OUT`), Movement Date, Quantity.

### Data Preparation (Power Query)

* Removed duplicate transaction logs and invalid delivery records.
* Standardized date and timestamp formats across order and movement tables.
* Transformed and cleaned movement status flags using `UPPER()` logic to ensure uniform filtering.
* Handled null values and unmapped warehouse identifiers using mapping rules.
* Created calculated columns for actual lead time in days, delivery status flags, and stock health indicators.

### Data Model (Star Schema)

![Spend Breakdown](https://github.com/tharannum/AL-Ghurair-Dashboard-/blob/main/datamodelling.png)

* **Fact Tables:**
* `Fact_PurchaseOrders` (PO commitments and vendor delivery performance)
* `Fact_Inventory` (Current snapshot of stock levels and valuation)
* `Fact_StockMovement` (Inbound and outbound warehouse transaction logs)


* **Dimension Tables:**
* `Dim_Calendar`
* `Dim_Suppliers`
* `Dim_Products`
* `Dim_Warehouses`


* **Relationships:** Active 1-to-Many ($1:*$) relationships built on primary/foreign surrogate keys (`SupplierID`, `ProductID`, `WarehouseID`, `Date`).

---

## 📈 3. Analysis Performed

### Power Query Transformations

* Cleaning and shaping procurement and WMS datasets.
* Setting up custom columns (`Lead Time Variance Days`, `On-Time Delivery Flag`, `Is Low Stock Flag`).

### DAX Measures

* `Total PO Value` = `SUM(Fact_PurchaseOrders[POValue])`
* `Total POs` = `COUNTROWS(Fact_PurchaseOrders)`
* `On Time Delivery Rate` = `DIVIDE(CALCULATE(COUNTROWS(Fact_PurchaseOrders), Fact_PurchaseOrders[IsOnTime] = 1), [Total POs], 0)`
* `Avg Actual Lead Time Days` = `AVERAGE(Fact_PurchaseOrders[ActualLeadTimeDays])`
* `Total Inventory Value` = `SUMX(Fact_Inventory, Fact_Inventory[QuantityOnHand] * Dim_Products[UnitCost])`
* `Total Stock On Hand` = `SUM(Fact_Inventory[QuantityOnHand])`
* `Low Stock Product Count` = `CALCULATE(DISTINCTCOUNT(Fact_Inventory[ProductID]), Fact_Inventory[QuantityOnHand] < RELATED(Dim_Products[ReorderLevel]))`
* `Stock Inflow` = `CALCULATE(SUM(Fact_StockMovement[Quantity]), UPPER(Fact_StockMovement[MovementType]) IN {"IN", "INBOUND"})`
* `Stock Outflow` = `CALCULATE(SUM(Fact_StockMovement[Quantity]), UPPER(Fact_StockMovement[MovementType]) IN {"OUT", "OUTBOUND"})`

### Visualizations

* **Header KPI Cards:** Instant high-level readouts for Spend (`AED 527K`), Orders (`25`), On-Time Delivery (`64%`), Lead Time (`8 Days`), Stock Value (`AED 588K`), and Low Stock Alerts (`4 SKUs`).
* **Line Chart:** `Monthly Spend Trend` comparing Order Date commitment vs. Delivery Date spend.
* **Donut Chart:** `Spend Distribution by Category` showing cost allocation across product lines.
* **Matrix / Tables:** `Supplier SLA & Performance Scorecard` and `Stockout Risk Alert Matrix` with dynamic low-stock filtering.
* **Horizontal Bar Chart:** `Inventory Capital Allocation by Warehouse Facility` highlighting locked-up cash.
* **Clustered Column Chart:** `Warehouse Operations Audit: Inbound Stock vs. Outbound Movement`.
* **Slicers:** Facility location buttons (Dubai, Abu Dhabi, Sharjah, RAK, KIZAD) and product category filters.

---

#### 1. Executive Procurement & Spend Optimization

##### **Q1.1: Where is our capital going? What is our total procurement spend, and how is it distributed across suppliers, product categories, and order statuses?**

* **Dashboard Visual Location:** Top Left KPI Card + Donut Chart + Top Right Supplier Matrix.
  ![Spend Breakdown](https://github.com/tharannum/AL-Ghurair-Dashboard-/blob/main/procurement.png)
* **Data-Driven Answer:**
* **Total Procurement Spend:** **AED 527K** across 25 Purchase Orders.
* **Category Distribution:** Spending is heavily concentrated in **Raw Materials (46.72% / AED 246.2K)**, followed by **Maintenance (22.71% / AED 119.7K)** and **Electrical (13.21% / AED 69.6K)**.
* **Supplier Spend Distribution:** **Apex Industrial Supplies** represents the largest financial outflow (**AED 111.2K across 7 POs**), followed by **Global Metals Trading (AED 102.3K)**.



##### **Q1.2: Are purchase volumes aligned with financial projections? How does our monthly purchase order (PO) spend compare across order dates vs. actual delivery dates?**

* **Dashboard Visual Location:** Bottom Right Line Chart (`Monthly Spend Trend`).
  ![Monthly Spend Trend](https://github.com/tharannum/AL-Ghurair-Dashboard-/blob/main/Monthly%20spend%20.png)
* **Data-Driven Answer:**
* Spend commitments spiked significantly between **July 2026 and August 2026**.
* The variance between `Total PO Value` (Order Date) and `Spend by Delivery Date` indicates an extended order-to-delivery lag during peak purchasing periods, impacting cash flow timing and inventory receipt schedules.



##### **Q1.3: Which product categories drive the highest procurement costs? Which high-cost SKUs contribute most to overall spending?**

* **Dashboard Visual Location:** `Spend Distribution by Category` Donut Chart + Supplier Detail.
  ![Category Spend Donut]( https://github.com/tharannum/AL-Ghurair-Dashboard-/blob/main/spend%20distribution%20.png)
* **Data-Driven Answer:**
* **Raw Materials** is the single largest cost driver, accounting for nearly half of total procurement capital (**46.72%**).
* Negotiating volume discounts on Raw Material SKUs with top vendors (such as Apex Industrial and Global Metals) offers the highest potential for spend optimisation.



---

#### 2. Supplier Performance & Reliability (SLA Tracking)

##### **Q2.1: Which suppliers are lagging behind on delivery times? What is the On-Time Delivery Rate (%) for each supplier?**

* **Dashboard Visual Location:** KPI Card 2 + `Supplier SLA & Performance Scorecard` Table.
* **Image Placeholder:** (https://github.com/tharannum/AL-Ghurair-Dashboard-/blob/main/matrix.png)
* **Data-Driven Answer:**
* **Overall On-Time Delivery (OTD) Rate:** **64%** across all supply lines.
* **Lagging Vendors:** **Global Metals Trading** and **Atlas Chemicals & Fluids** have severe delays with only a **50% OTD Rate**.
* **Top Performers:** **Gulf Packaging Solutions** and **Al Ghurair Packaging Div** achieved a **100% OTD Rate**.



##### **Q2.2: How accurate are our contractual lead times? What is the average lead time variance across suppliers?**

* **Dashboard Visual Location:** KPI Card 4 + `Avg Actual Lead Time` column in Supplier Table.
![Lead Time Variance](https://github.com/tharannum/AL-Ghurair-Dashboard-/blob/main/lead%20time.png)
* **Data-Driven Answer:**
* **Average Lead Time:** **8 Days** across all fulfillment lines.
* **Worst Lead-Time Offender:** **EuroMetal Tech GmbH** averages **21 Days** per delivery, representing a severe operational bottleneck despite a smaller total PO count.



##### **Q2.3: Which suppliers represent a single-point-of-failure risk?**

* **Dashboard Visual Location:** Supplier Performance Matrix.
  ![Vendor Concentration](https://github.com/tharannum/AL-Ghurair-Dashboard-/blob/main/matrix.png)
* **Data-Driven Answer:**
* **Apex Industrial Supplies** holds **28% of total purchase orders (7 out of 25 POs)** and **AED 111.2K in spend**.
* A delivery failure from Apex Industrial creates a significant operational risk for production lines due to vendor reliance.



---

### 📄 Page 2: Inventory Valuation & Warehouse Operations Audit

> **Dashboard Snapshot (Page 2):**
> ![Page 2 Overview](./images/page2_inventory_operations.png)
> *(Insert your Page 2 full screenshot here)*

---

#### 3. Inventory Management & Supply Chain Risk

##### **Q3.1: What is our total locked-up capital in stock across all warehouses?**

* **Dashboard Visual Location:** Page 2 Header KPI Card 1.
![Inventory Valuation KPI](https://github.com/tharannum/AL-Ghurair-Dashboard-/blob/main/total%20inventory.png)
* **Data-Driven Answer:**
* **Total Locked-Up Inventory Value:** **AED 588K** across 6,000 total physical units on hand.



##### **Q3.2: Which products are at risk of stockouts? Which SKUs have fallen below their designated Reorder Level?**

* **Dashboard Visual Location:** Page 2 Top Right Matrix (`Stockout Risk Alert`).
![Low Stock Matrix](https://github.com/tharannum/AL-Ghurair-Dashboard-/blob/main/low%20stock.png)
* **Data-Driven Answer:**
* **4 Critical SKUs** have breached safety stock thresholds and require immediate purchase orders.
* **Example Flagged Item:** *Industrial Lubricant 5L* (Maintenance Category) has **47 units on hand** against a designated **Reorder Level of 50 units**.



##### **Q3.3: Where is inventory geographically concentrated?**

* **Dashboard Visual Location:** Bottom Right Bar Chart (`Inventory Capital Allocation by Warehouse Facility`).
![Warehouse Valuation Bar Chart](./images/warehouse_capital_bar_chart.png)`
* **Data-Driven Answer:**
* Capital is heavily concentrated in **KIZAD Industrial Zone (AED 210K)** and **Jebel Ali Logistics Hub (AED 170K)**, which together represent over **64% of total inventory valuation**.
* **RAK Industrial Park** holds the lowest stock concentration at **AED 50K**.



---

#### 4. Warehouse Operations & Stock Movement Auditing

##### **Q4.1: How fast is stock moving through our facilities? What is the ratio of Stock Inflow vs. Stock Outflow?**

* **Dashboard Visual Location:** Bottom Left Stacked Bar Chart (`Warehouse Operations Audit: Inbound Stock vs. Outbound Movement`).
* **Image Placeholder:** `![Warehouse Stock Movement](./images/stock_inflow_outflow_chart.png)`
* **Data-Driven Answer:**
* **Total Physical Movement:** **2,000 Units Inflow** processed across facilities.
* Facilities like **Al Sajaa Facility (95.89% Inflow ratio)** and **KIZAD Industrial Zone (82.35% Inflow ratio)** act primarily as holding locations, while **RAK Industrial Park (100% Outflow)** operates as an active fulfilment point.



##### **Q4.2: Which items are fast-moving vs. slow-moving?**

* **Dashboard Visual Location:** Interaction between Low Stock Matrix and Movement Audit.
* **Image Placeholder:** `![Fast Slow Moving SKUs](./images/sku_velocity_analysis.png)`
* **Data-Driven Answer:**
* **Fast-Moving SKUs:** Maintenance and packaging materials show high throughput ratios and quick depletion below safety limits.
* **Slow-Moving SKUs:** Specific electrical component categories remain stored in high valuations at KIZAD without proportional outbound velocity.

---

## 🔍 5. Insights & Interpretation

### Insight 1 — Raw Materials Drive Nearly Half of Total Procurement Spend

Total procurement spend stands at **AED 527K** across 25 Purchase Orders. **Raw Materials** accounts for **46.72% (AED 246.2K)** of total capital allocation, followed by Maintenance (22.71% / AED 119.7K) and Electrical (13.21% / AED 69.6K).

### Insight 2 — Low Overall Supplier SLA Compliance (64% OTD Rate)

The organization-wide On-Time Delivery rate is **64%**, meaning 36 out of 100 purchase orders arrive late. Suppliers such as **Global Metals Trading** and **Atlas Chemicals & Fluids** show a poor **50% OTD Rate**, directly causing operational delays downstream.

### Insight 3 — High Vendor Concentration & Single-Point-of-Failure Risk

**Apex Industrial Supplies** represents the single largest spend concentration at **AED 111.2K across 7 Purchase Orders** (28% of total order volume). Any supply chain disruption with Apex Industrial presents a critical operational vulnerability.

### Insight 4 — Severe Lead-Time Offender Identified

While average lead time across all vendors sits at **8 Days**, **EuroMetal Tech GmbH** averages **21 Days** per delivery—more than double the company average—despite fulfilling only 2 POs.

### Insight 5 — AED 588K Tied Up in Physical Inventory

Total current inventory valuation across all UAE warehouses is **AED 588K** (6,000 units on hand). Capital is heavily concentrated in two major facilities: **KIZAD Industrial Zone (AED 210K)** and **Jebel Ali Logistics Hub (AED 170K)**, representing over **64%** of total inventory capital.

### Insight 6 — 4 Critical SKUs Breached Safety Reorder Thresholds

The Stockout Risk Alert system flagged **4 critical SKUs** below minimum safety stock levels. For example, **Industrial Lubricant 5L** in the Maintenance category dropped to **47 units on hand** against its safety **Reorder Threshold of 50 units**, requiring immediate PO placement to prevent facility downtime.

### Insight 7 — Imbalanced Warehouse Flow Rates

Stock inflow totals **2,000 units** across facilities. **Al Sajaa Facility** operates primarily as an inbound storage location (**95.89% Inflow ratio**), whereas **RAK Industrial Park** operates heavily on the outbound distribution side (**100% Outflow ratio**).

---

## 📌 6. Recommendations

### Procurement & Commercial Team

* **Vendor Contract Penalties:** Enforce financial penalty clauses or renegotiate terms with vendors exhibiting <60% OTD rates (**Global Metals Trading** and **Atlas Chemicals**).
* **Supplier Diversification:** Reduce dependency on **Apex Industrial Supplies** (28% of total PO volume) by onboarding secondary local vendors to mitigate single-point-of-failure risks.

### Supply Chain & Inventory Operations

* **Automated Reorder Triggers:** Establish automated purchase requisitions for the 4 flagged low-stock SKUs (including *Industrial Lubricant 5L*) whenever stock drops within 5% of designated Reorder Levels.
* **Regional Stock Rebalancing:** Reallocate slow-moving inventory held at **KIZAD Industrial Zone (AED 210K)** to high-outflow fulfillment centers like **RAK Industrial Park** to optimize localized stock availability.

### Executive Leadership

* **Target Lead-Time Thresholds:** Establish a strict corporate KPI enforcing maximum vendor lead times of **10 days**, flagged specifically for suppliers operating overseas like EuroMetal Tech.
* **Quarterly Capital Optimization:** Target a 15% reduction in locked-up inventory value in non-core facilities by reviewing slow-moving categories quarterly.

---

## 🛠️ 7. Skills Used

* **Power BI Desktop:** Multi-page dashboard layout, custom page navigation buttons, UI rounding, and brand theme styling.
* **Data Modeling:** Star Schema design (`Fact_PurchaseOrders`, `Fact_Inventory`, `Fact_StockMovement`, `Dim_Suppliers`, `Dim_Products`, `Dim_Warehouses`, `Dim_Calendar`).
* **DAX Development:** Time intelligence, conditional alert measures (`Is Low Stock`), aggregations, and performance metrics (`On Time Delivery Rate %`, `Stock Inflow/Outflow`).
* **Power Query (M Language):** Data cleaning, null handling, data type standardization, and conditional flag creation.
* **Supply Chain & Procurement Analytics:** Vendor SLA tracking, lead-time variance analysis, inventory valuation, stockout risk mitigation, and warehouse material flow modeling.
* **Analytical Storytelling:** Translating raw operational logs into executive-level insights and recommendations.

---

## 🏁 8. Project Outcome

This project delivers:

* A fully interactive, 2-page executive Power BI Dashboard Suite (**Page 1: Procurement & Supplier Performance Analytics**, **Page 2: Inventory Valuation & Warehouse Operations Audit**).
* A star-schema data model connecting financial procurement spend with physical warehouse logistics.
* Automated dynamic low-stock alerts and vendor SLA scorecards powered by custom DAX measures.
* Clear, decision-ready insights and strategic recommendations for Al Ghurair's business, logistics, procurement, and executive leadership teams.
