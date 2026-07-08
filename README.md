# ☕ Starbucks Daily Sales Analysis Dashboard
**An end-to-end SQL + Power BI project turning raw PostgreSQL transaction data into an hourly sales performance dashboard.**

[Home Page](https://github.com/abhaykumar-kotval/Daily-Sales-Starbucks-Analysis-Dashboard/blob/main/IMG%201.PNG)

---

## 📌 Short Description / Purpose

This project is a full-stack data analytics build: a **normalized relational database in PostgreSQL** feeding a **live-connected Power BI dashboard** that tracks daily Starbucks sales performance by hour, by customer, and by product. Rather than starting from a flat, pre-cleaned spreadsheet, the project begins at the database layer — designing tables, primary/foreign keys, and relationships in SQL — before connecting Power BI directly to the database and layering on DAX-driven KPIs, dynamic conditional formatting, and a transaction-level drill-through table.

The dashboard is built for a **store operations / sales manager audience** who needs to answer a specific operational question: *"At what hour are we performing best (and worst), and who are we selling to?"* This is the kind of view a shift or regional manager would check throughout the day to spot the peak order window, monitor performance against a daily target, and drill into individual transactions when something needs investigating — demonstrating not just chart-building but the ability to design the database schema the reporting layer depends on.

---

## 🛠️ Tech Stack

| Technology | How it was used |
|---|---|
| **PostgreSQL** | Hosts the source database (`starbucks`). Three relational tables — `customers`, `items`, and `sales` — were created via `CREATE TABLE` statements, seeded with `INSERT` statements, and linked with explicit foreign key constraints (`fk_customer`, `fk_item`). |
| **SQL** | Used for schema design (primary keys, data types), referential integrity (foreign keys tying `sales.customer_id` → `customers.customer_id` and `sales.item_id` → `items.id`), and the seed data itself. |
| **Power BI Desktop** | Connects live to PostgreSQL (`localhost:5432`, database `starbucks`) to build the report. Handles data modeling, DAX measures, and the three-page report (Home, Overview, Details). |
| **Power BI — Direct/Live Connection** | The report was configured as a live connection into PostgreSQL, rather than an imported static extract — meaning the dashboard reflects the database as of each refresh, keeping the analytics layer separate from the data layer. |
| **DAX** | A dedicated `_Measures` table holds all KPI, target, and conditional-formatting logic (see Data Model section) — including dynamic "peak hour" highlighting measures that recolor a chart's bars based on which hour is performing best. |
| **Data Modeling (Star-Schema-style)** | A `Calender` table generated via `SUMMARIZE(sales, sales[datetime])` acts as the time dimension, with derived `Hour` and `Hours_Sort` columns, connected to the `sales` fact table alongside the `customers` and `items` dimension tables. |

---

## 🗂️ Data Source

The data originates from a **hand-built relational schema in PostgreSQL**, defined and seeded directly in SQL (see `Steps.txt` in this repo for the full DDL/DML):

- **`customers`** (20 records) — `customer_id` (PK), name, email, phone, age, gender.
- **`items`** (20 records) — `id` (PK), item name, and nutritional attributes (`calories`, `fat`, `carb`, `fiber`, `protein`), plus a `type` field distinguishing **Beverage** vs. **Food** items (e.g., Caffe Latte, Cappuccino, Espresso, Blueberry Muffin, Chicken Sandwich).
- **`sales`** (100 transaction records, `transaction_id` PK) — the fact table, capturing `store_id`, `datetime`, `customer_id` (FK), `item_id` (FK), `quantity`, `price`, `total_amount`, `payment_mode` (UPI / Card / Cash), and `customer_type` (New / Regular).
- Foreign keys explicitly enforce that every sale ties back to a valid customer and item (`fk_customer`, `fk_item`), which is good relational-database practice and reflects real schema design rather than a flattened export.

**Structure & connection:** Power BI connects to this database as a **Direct/live PostgreSQL connection** (per the documented connection steps: `localhost:5432`, database `starbucks`), importing the `sales`, `customers`, and `items` tables and building a `Calender` table inside Power BI itself via DAX.

**Scale & limitations:**
- The dataset is a **compact demo set** — 20 customers, 20 menu items, and 100 transactions spanning two dates (19–20 April 2026) — clearly built to exercise the full pipeline (SQL → relationships → Power BI → DAX) rather than to represent a full year of live retail volume.
- Because all transactions fall within a two-day window, the "by hour" analysis is meaningful, but there is no multi-week/monthly trend view in this model — that would require a longer date range in the `sales` table.
- Only a single `store_id` field exists in the fact table with no accompanying `stores` dimension table, so store-level comparison isn't currently modeled, only hour/customer/item-level analysis.

---

## ✨ Features / Highlights

### 🎯 Business Problem

Retail and QSR (quick-service restaurant) managers need to know **when** their store is busiest and **whether** they're on pace to hit daily targets — not just what happened at the end of the day, but which specific hours are driving (or dragging down) performance. Without this, staffing, inventory prep, and promotional timing decisions are made on gut feel rather than evidence. This dashboard directly answers: *which hour generates the most revenue, orders, and volume — and are we tracking toward today's targets?*

### 🎯 Goal of the Dashboard

The report is built for **intraday monitoring and target-tracking**, structured across three purpose-built pages:
- **Home** — a branded landing/navigation page.
- **Overview** — the executive snapshot: today's totals, progress against targets, and hour-by-hour performance trends.
- **Details** — a transaction-level table for drill-down/investigation.

This progression (branding → summary → detail) is a deliberate UX pattern that lets a manager glance at the Overview for a quick pulse check, then drop into Details only when they need to investigate a specific customer or transaction.

### 🎯 Walkthrough of Key Visuals

**🏠 Home Page**

![Home Page](IMG_1.PNG)

A branded landing screen with the Starbucks logo and a simple top navigation bar (**Home / Overview / Details**) built with Power BI's **page navigator** visual. This exists purely for polish and usability — giving the report a "product" feel rather than a raw set of Power BI tabs, and giving users an explicit, guided way to move between the summary and detail views.

**📊 Overview Page**

![Overview Page](IMG_2.PNG)

*🔢 KPI Ring Cards (Order Count, Customer Count, Total Amount, Total Quantity)*
Four headline numbers — **100 orders, 20 customers, ₹33.22K total amount, 158 items sold** — each paired with a **donut chart used as a progress ring** rather than a plain KPI card. Each ring plots the actual measure (`Order_Count`, `Customer_Count`, `Total_Amount`, `Total_Quantity`) against a corresponding DAX target constant (`Total_Order_Target = 300`, `Customer_Order_Target = 70`, `Total_Amount_Target = 60000`, `Total_Quantity_Target = 300`). This turns a plain number into an instantly readable **"how close are we to today's goal"** signal — far more actionable for a manager than the raw total alone.

*📈 Avg Order Amount by Hour (Clustered Column Chart)*
Plots the `Avg_Order_Amount` DAX measure across every hour of the day (8 AM–10 PM), revealing that **6 PM is the strongest hour for average order value** in this sample (₹488), highlighted in bright green while every other hour renders gray. That highlight isn't manual formatting — it's driven by the `Avg_Order_FX` measure, which compares each hour's average against `Avg_Order_Max` (the best-performing hour, computed via `MAXX(ALL(Calender[Hour], Calender[Hours_Sort]), [Avg_Order_Amount])`) and returns a highlight color only when the two match. This is a clean example of **dynamic conditional formatting driven entirely by DAX**, so the "best hour" callout updates automatically as data changes — no manual color-coding required.

*📈 Total Amount by Hour (Clustered Column Chart)*
Same pattern applied to total revenue: **8 AM stands out as the top revenue hour** (₹3.6K), auto-highlighted via the equivalent `Total_Amount_FX` / `Total_Amount_Max` measure pair. This visual answers a different question than the Avg Order chart — it's about *volume of revenue*, not order size — and together the two charts let a manager distinguish "we sold a lot" from "customers spent more per order."

*📈 Total Quantity by Hour (Clustered Column Chart)*
The same highlight-the-best-hour pattern applied to unit volume, flagging **10 AM as the peak hour for items sold** (16 units) via `Quantity_FX` / `Quantity_Max`. This is the operational chart most relevant to **staffing and prep planning** — knowing the peak unit-volume hour (not just peak revenue hour) tells a manager when extra hands or pre-made stock matter most.

*🕒 Last Update Timestamp Card*
A single text card showing *"Last Update – 20-Apr-26 : 10:20 PM"*, generated by the `Formate_Max_Date` measure (`"Last Update - " & FORMAT([Max_Date], "dd-mmm-yy : hh:mm AM/PM")`). This is a small but important piece of report craftsmanship — it tells any viewer exactly how current the data is, which matters for an intraday operational dashboard where stale data could lead to the wrong staffing or restocking decision.

**📋 Details Page**

![Details Page](IMG_3.PNG)

A full drill-down table joining **`customers`, `items`, and `sales`** — showing `transaction_id`, `customer_name`, `customer_phone`, `item`, `datetime`, `customer_type`, `payment_mode`, alongside the `Total_Amount`, `Total_Quantity`, and `Order_Count` measures, with a grand total row (₹33,220.00 / 158 units / 100 orders). This is the page a manager or analyst uses when the Overview raises a question — e.g., "who exactly bought during that 6 PM peak?" — turning the dashboard from a passive report into an investigable one.

### 💼 Business Impact & Insights

- **Target tracking made visual:** Converting flat KPI numbers into progress rings against explicit DAX-defined targets (e.g., ₹60,000 revenue target vs. ₹33.22K achieved) gives an at-a-glance read on daily pacing rather than requiring the viewer to do the math themselves.
- **Data-driven staffing signals:** Automatically flagging the peak hour for **average order value (6 PM)**, **total revenue (8 AM)**, and **unit volume (10 AM)** as three *different* hours is itself an insight — it shows the store doesn't have one single "peak hour" but three distinct peaks depending on which metric matters, which has direct implications for how staffing and prep schedules should be shaped throughout the day.
- **Investigation-ready design:** By joining `customers`, `items`, and `sales` into a single detail table with running totals, the report supports root-cause drill-down (e.g., verifying which specific items or customers drove a given hour's numbers) without needing to go back to the database.
- **Reusable, scalable schema:** Because the underlying SQL schema uses proper primary/foreign key relationships rather than a flat file, this project demonstrates a foundation that could realistically scale to more stores, more menu items, or a longer transaction history without redesigning the data model — only the `Calender` table and DAX targets would need adjustment.

---
