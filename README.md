# 🏨 AtliQ Hospitality Analysis: Hotel Revenue & Occupancy Dashboard



> An interactive Power BI dashboard that tracks revenue, occupancy, ADR, RevPAR, cancellations and guest ratings across AtliQ's hotel properties in four Indian cities.



## 🔹 Overview
AtliQ Hospitality operates **25 properties** across **Delhi, Mumbai, Bangalore and Hyderabad** in two categories (**Luxury** and **Business**). This project turns raw booking data into a dashboard that lets management monitor performance, compare properties and understand booking behavior by platform, room class and time period.

**Analysis period:** 1 May 2022 – 31 July 2022 (92 days)

## ❓ Business Problem
Hotel management needs a single view to answer questions such as:
- How much revenue is each property, city and category generating?
- How well are rooms being filled (occupancy) and at what price (ADR, RevPAR)?
- How much revenue is lost to cancellations and no-shows?
- Which booking platforms perform best?
- How do weekdays compare with weekends, and how do trends move by month and week?

## 📂 Dataset
Five CSV files loaded through Power Query:

| Table | Type | Rows | Description |
|-------|------|------|-------------|
| `fact_bookings` | Fact | 134,590 | One row per booking: dates, guests, room category, platform, rating, status, revenue generated and realized |
| `fact_aggregated_bookings` | Fact | 9,200 | Daily successful bookings and capacity by property and room category |
| `dim_hotels` | Dimension | 25 | Property ID, name, category, city |
| `dim_rooms` | Dimension | 4 | Room class: Standard, Elite, Premium, Presidential |
| `dim_date` | Dimension | 92 | Date, month, week number, weekday/weekend type |

## 🧩 Data Model
Star schema with single-direction, many-to-one relationships:

```
dim_hotels ──< fact_bookings >── dim_rooms
    │               │
    │               └──> dim_date (check_in_date)
    └──< fact_aggregated_bookings >── dim_rooms
                    └──> dim_date (check_in_date)
```

| From (many) | To (one) | Key |
|-------------|----------|-----|
| `fact_bookings` | `dim_hotels` | `property_id` |
| `fact_bookings` | `dim_rooms` | `room_category` → `room_id` |
| `fact_bookings` | `dim_date` | `check_in_date` → `date` |
| `fact_aggregated_bookings` | `dim_hotels` | `property_id` |
| `fact_aggregated_bookings` | `dim_rooms` | `room_category` → `room_id` |
| `fact_aggregated_bookings` | `dim_date` | `check_in_date` → `date` |

All measures live in a dedicated **`Key_Measure`** table.

## 📐 KPIs & DAX Measures

| Measure | DAX | Meaning |
|---------|-----|---------|
| Revenue | `SUM(fact_bookings[revenue_generated])` | Total revenue generated |
| Total_bookings | `COUNT(fact_bookings[booking_id])` | All bookings |
| Total_Capacity | `SUM(fact_aggregated_bookings[capacity])` | Total available room nights |
| Occupancy % | `DIVIDE([Total_Successful_bookings],[Total_Capacity])` | Share of capacity filled |
| ADR | `DIVIDE([Revenue],[Total_bookings],0)` | Average daily rate |
| RevPAR | `DIVIDE([Revenue],[Total_Capacity])` | Revenue per available room |
| Cancellation % | `DIVIDE([Total_Cancelled_Booking],[Total_bookings])` | Share of bookings cancelled |
| NoShow Rate % | `DIVIDE([Total_NoShow_Bookings],[Total_bookings])` | Share of no-shows |
| Realisation % | `1-([Cancellation %]+[NoShow_Rate%])` | Share of bookings that were fulfilled |
| Average Ratings | `AVERAGE(fact_bookings[ratings_given])` | Mean guest rating |
| DBRN | `DIVIDE([Total_bookings],[No_Of_Days])` | Daily booked room nights |
| DSRN | `DIVIDE([Total_Capacity],[No_Of_Days])` | Daily sellable room nights |
| DURN | `DIVIDE([Total_checked_out],[No_Of_Days])` | Daily utilized room nights |
| Booking % by Platform | `DIVIDE([Total_bookings], CALCULATE([Total_bookings], ALL(...)))*100` | Platform share of bookings |
| Booking by Room Class | same pattern over `dim_rooms[room_class]` | Room class share of bookings |

Supporting measures: `Total_Successful_bookings`, `Total_Cancelled_Booking`, `Total_checked_out`, `Total_NoShow_Bookings`, `No_Of_Days`.

## 📊 Dashboard Pages

| Page | Contents |
|------|----------|
| **Trends** | KPI cards (Revenue, Occupancy %, Avg Ratings, Realisation %, DSRN, ADR); occupancy by property category (donut); platform-wise Realisation % and ADR (combo chart); RevPAR, ADR and Occupancy % trends by month and week (line chart); weekday vs weekend table |
| **Key Matrics** | Property-level table (revenue, RevPAR, occupancy, realisation, cancellation %, ADR, DBRN/DSRN/DURN, ratings, capacity) and a booking-trend chart (total, successful and cancelled bookings by property) |
| **Tooltip pages** | Revenue by week/month, Occupancy by room class, and Average Ratings by property, shown on hover |

**Slicers on both main pages:** Property, City, Booking Status, Platform, Month, Week.

## 📈 Key Insights
Figures below are computed from the data model for the full May–July 2022 period.

| KPI | Value |
|-----|-------|
| Total Revenue | **₹2.01B** (₹2,007.5M) |
| Total Bookings | **134,590** |
| Occupancy | **57.87%** |
| ADR | **₹14,916** |
| RevPAR | **₹8,632** |
| Realisation | **70.15%** |
| Cancellation Rate | **24.83%** |
| No-Show Rate | **5.02%** |

- **Mumbai leads revenue** (~₹785M), followed by Bangalore (~₹495M), Hyderabad (~₹381M) and Delhi (~₹346M)
- **Luxury properties generate ~62% of revenue** (~₹1.24B vs ~₹0.77B for Business)
- **Atliq Exotica** is the top revenue property (~₹375M); **Atliq Seasons** is the lowest (~₹78M) and also has the lowest occupancy (44.6%)
- **Atliq Blu** has the highest occupancy (62.0%)
- **Delhi has the highest city occupancy** (60.5%); Bangalore the lowest (55.8%)
- **Roughly 30% of bookings are lost** to cancellations (24.8%) and no-shows (5.0%), the biggest revenue leakage point
- **"Others" is the largest booking channel** (~41%), followed by MakeYourTrip (~20%) and LogTrip (~11%)
- **ADR is very consistent** across platforms and between weekdays (₹14,924) and weekends (₹14,903), suggesting little dynamic pricing
- **Occupancy is similar across room classes** (57.6%–59.2%), with Presidential slightly highest


## 📝 Data Notes
- **Average Ratings** uses `AVERAGE` over all rows, and about 58% of bookings (77,907) have a rating of `0`, which appears to mean "not rated". The dashboard shows **1.52**; excluding zeros the average is **3.62**. Consider treating 0 as blank.
- In `fact_aggregated_bookings`, `successful_bookings` totals 134,590, the same as total bookings including cancelled and no-show records, so **Occupancy %** may be overstated if "successful" should exclude those.
- `dim_date` contains the typo `weekeday` in the `day_type` column, and the page name is spelled "Key Matrics".

## 💡 Recommendations
1. **Reduce cancellations and no-shows** with stricter policies, deposits or reminders, since they put about 30% of bookings at risk
2. **Investigate Atliq Seasons**, which has the lowest occupancy, ratings and revenue
3. **Introduce dynamic pricing**: ADR is flat across weekdays, weekends and platforms
4. **Grow direct channels**, which make up only ~15% of bookings, to reduce platform dependence
5. **Replicate Atliq Blu and Delhi practices** to lift occupancy elsewhere
6. **Fix the rating logic** so guest satisfaction is measured accurately

## ▶️ How to Run
1. Clone this repository
2. Open `AtliQ_Hospitality.pbix` in **Power BI Desktop**
3. In **Transform data → Data source settings**, repoint the CSV paths to your local `data/` folder (the file currently reads from a local `Documents` path)
4. Click **Refresh**

## 📁 Repository Structure
```
atliq-hospitality-analysis/
├── data/               # dim_date, dim_hotels, dim_rooms, fact_bookings, fact_aggregated_bookings (CSV)
├── AtliQ_Hospitality.pbix
├── images/             # Dashboard screenshots
└── README.md
```
