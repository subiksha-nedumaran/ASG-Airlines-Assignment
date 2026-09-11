# ASG Airlines – End-to-End Data Engineering Case Study

## 1. Project Overview

ASG Airlines is a nationwide airline that operates flights across multiple cities through its booking, scheduling and payment systems.

This project implements an end-to-end data engineering pipeline using Databricks Free Edition and Microsoft Power BI. The solution performs data ingestion, profiling, data-quality validation, cleaning, standardization, privacy protection, Silver-layer transformation and Gold-layer analytical modeling.

The final Gold datasets are used to build an interactive four-page Power BI dashboard for operational, duration, airline, revenue and data-quality analysis.

---

## 2. Objectives

- Ingest airline data from the source Excel workbook.
- Profile the incoming datasets.
- Identify invalid records, missing values and duplicates.
- Clean and standardize the datasets.
- Validate flight identifiers and business keys.
- Calculate and validate flight duration.
- Handle overnight flights correctly.
- Protect passenger-related sensitive information.
- Create Silver and Gold analytical datasets.
- Generate operational and financial KPIs.
- Build an interactive Power BI dashboard.
- Document architecture, data flow, data model, privacy and access-control strategy.

---

## 3. Technology Stack

| Component | Technology |
|---|---|
| Data Processing | Python / Pandas |
| Data Engineering | Databricks Free Edition |
| Storage | Databricks Volumes |
| Source | Excel Workbook |
| Visualization | Microsoft Power BI |
| Version Control | GitHub |
| Privacy | SHA-256 hashing and masking |

> The implemented pipeline uses Databricks Free Edition. Azure Databricks and ADLS Gen2 are described as the recommended production deployment architecture, but were not used as the implementation environment for this project.

---

## 4. Source Data

The source workbook contains four datasets:

- Flights
- Bookings
- Passengers
- Payments

### Raw Data Counts

| Dataset | Records |
|---|---:|
| Flights | 1,020 |
| Bookings | 1,000 |
| Passengers | 1,039 |
| Payments | 1,000 |

---

## 5. End-to-End Architecture


                 ASG AIRLINES
                      |
                      v
             Source Excel Workbook
                      |
                      v
              Databricks Ingestion
                      |
                      v
          Profiling & Data Quality Checks
                      |
                      v
            Cleaning & Standardization
                      |
                      v
              Privacy Transformation
                      |
                      v
                SILVER LAYER
                      |
                      v
          Fact & Dimension Construction
                      |
                      v
                  GOLD LAYER
                      |
                      v
                  Power BI
                      |
                      v
          Interactive Business Dashboard

##6 Data Flow Diagram
Level 0 - Context
Source Airline Data
        |
        v
ASG Airlines Data Engineering Pipeline
        |
        v
Sanitized Gold Analytical Data
        |
        v
Power BI
        |
        v
Business Users

Level 1 – Pipeline Data Flow
Excel Source
    |
    v
Data Ingestion
    |
    v
Raw Profiling
    |
    v
Data Quality Checks
    |
    v
Cleaning & Standardization
    |
    v
Privacy Transformation
    |
    v
Silver Data
    |
    v
Fact / Dimension Modeling
    |
    v
Gold Data
    |
    v
Power BI Dashboard

##7 Raw → Silver → Gold Processing
Raw Layer

The original Excel workbook is loaded and profiled before transformation.

Silver Layer

The Silver layer contains cleaned and standardized datasets.

Major transformations include:

Text standardization.
Missing-value handling.
Duplicate detection and removal.
Flight ID validation.
Flight business-key enforcement.
Booking-status normalization.
Payment validation.
Flight-duration calculation.
Overnight-flight detection.
Duration anomaly detection.
Passenger PII protection.
Gold Layer

The Gold layer contains analytical fact and dimension datasets:

fact_flights
fact_bookings
fact_payments

dim_route
dim_airline
dim_date
dim_passenger

##8. Data Cleaning and Quality Strategy
Flight Data
Standardized text fields.
Missing airline information was handled where possible.
Exact duplicate rows were removed.
Duplicate flight IDs were identified.
flight_id was treated as the flight business key.
The earliest departure record was retained when duplicate flight IDs were encountered.
Flight duration was calculated from departure and arrival timestamps.
Invalid duration calculations were corrected where required.
Overnight flights were handled using the departure and arrival dates.
Duration anomalies were flagged.
Booking Data
Booking fields were standardized.
Booking IDs were validated.
Booking statuses were normalized.
Invalid/unrecognized booking statuses were mapped to UNKNOWN.
Booking records were linked to flights.
Payment Data
Payment records were validated.
Invalid or missing payment records were flagged.
Revenue calculations use only valid payments.

##9. Flight Duration and Overnight Handling

Flight duration is calculated from departure and arrival timestamps.

For flights where the arrival occurs on the following calendar day, the pipeline correctly handles the date transition rather than treating the arrival time as occurring before departure.

The pipeline also identifies overnight flights using the departure and arrival dates.

Final results:

Average flight duration: 164.54 minutes
Overnight flights: 122
Duration corrections: 1
Extreme duration anomalies: 0

##10. Data Quality Results
| Data Quality Metric              | Result |
| -------------------------------- | -----: |
| Raw Flights                      |  1,020 |
| Clean Flights                    |  1,004 |
| Duplicate flight records removed |     16 |
| Invalid flight IDs               |      0 |
| Duration corrections             |      1 |
| Overnight flights                |    122 |
| Extreme duration anomalies       |      0 |
| Bookings                         |  1,000 |
| Valid payments                   |    922 |
| Invalid / missing payments       |     78 |

##11. Gold Data Model
Fact Tables
fact_flights

Contains one cleaned analytical record per flight business key.

Important attributes include:

flight_id
airline
source
destination
departure_time
arrival_time
duration_minutes
is_overnight
duration_corrected
duration_anomaly
fact_bookings

Contains booking-level analytical information.

Important attributes include:

booking_id
flight_id
protected passenger key
booking date
booking status
status flags
fact_payments

Contains payment-level analytical information.

Important attributes include:

payment_id
booking_id
payment method
amount
payment validity
Dimension Tables
dim_route – 30 routes
dim_airline – 6 airlines
dim_date – 369 dates
dim_passenger – protected passenger information
Power BI Relationships
fact_flights[flight_id]
        1
        |
        *
fact_bookings[flight_id]


fact_bookings[booking_id]
        1
        |
        *
fact_payments[booking_id]

##12. Privacy Protection

Passenger-related sensitive information is protected during transformation.

Implemented Controls
Passenger IDs are hashed using SHA-256.
Passport numbers are hashed.
Emergency-contact names are masked.
Emergency-contact phone numbers are hashed.
Raw passenger IDs are removed from the analytical booking fact.
Raw passenger PII is not exposed in the analytical Gold model used for reporting.

This reduces unnecessary exposure of sensitive passenger information.

##13. Access Control

The solution follows the principle of least privilege.

Recommended Production Access Model
| Role             | Raw        | Silver     | Gold       |
| ---------------- | ---------- | ---------- | ---------- |
| Data Engineer    | Read/Write | Read/Write | Read/Write |
| Business Analyst | No Access  | No Access  | Read       |
| Power BI User    | No Access  | No Access  | Read       |

The current Databricks Free Edition workspace did not provide an appropriate individual/group principal for implementing this separation. Therefore, broad All account users access was intentionally not granted.

In a production Azure Databricks deployment, Unity Catalog and role-based access control would be used to enforce this separation.

##14. Key Business KPIs
| KPI                        |         Result |
| -------------------------- | -------------: |
| Total Flights              |          1,004 |
| Average Flight Duration    | 164.54 minutes |
| Overnight Flights          |            122 |
| Total Bookings             |          1,000 |
| Confirmed Bookings         |            320 |
| Cancelled Bookings         |            314 |
| Pending Bookings           |            291 |
| Unknown Bookings           |             75 |
| Valid Payments             |            922 |
| Invalid / Missing Payments |             78 |
| Total Revenue              |   7,385,142.98 |
| Average Payment Amount     |       8,009.92 |

##15. Power BI Dashboard
The Power BI report contains four pages.

Page 1 – Executive Operations Dashboard

Includes:

Total Flights
Average Flight Duration
Total Bookings
Total Revenue
Overnight Flights
Flight distribution by airline
Route traffic
Booking status distribution
Airline slicer
Source slicer
Destination slicer
Page 2 – Duration Analysis

Includes:

Average Flight Duration
Overnight Flights
Duration Corrections
Average Duration by Airline
Top 10 Routes by Average Duration
Overnight vs Non-Overnight Flights
Page 3 – Airline & Revenue Trends

Includes:

Flight distribution by airline
Bookings by airline
Revenue by payment method
Payment validation status
Source-to-destination traffic matrix
Airline flight-volume treemap
Page 4 – Delay & Anomaly Insights

Includes:

Overnight Flights
Duration Corrections
Corrected-flight details
Duration anomaly status
Data-quality indicators
Duplicate flight records removed
Invalid/missing payment indicators

The source data does not contain a dedicated delay-minutes field. Therefore, the project does not fabricate a delay metric. The fourth page focuses on duration corrections, duration anomalies, overnight operations and data-quality indicators.

##16. Assumptions
flight_id is treated as the flight business key.
For duplicate flight IDs, the earliest departure record is retained.
Overnight flights are determined using departure and arrival dates.
Invalid booking statuses are mapped to UNKNOWN.
Revenue KPIs include only valid payments.
The source data does not contain a dedicated delay-minutes field.
Passenger-sensitive information is protected before analytical reporting.
Power BI reporting uses sanitized Gold-layer data.

##17. Scalability and Production Considerations

For production-scale deployment, the solution can be extended with:

Azure Data Lake Storage Gen2 for cloud storage.
Azure Databricks for distributed processing.
Delta Lake tables for reliable analytical storage.
Unity Catalog for governance and access control.
Incremental data ingestion.
Databricks Jobs for scheduled execution.
Pipeline monitoring and failure alerts.
Power BI Service for report distribution.

The current implementation is a working case-study implementation using Databricks Free Edition.

##18. Repository Contents
ASG-Airlines-Assignment/
│
├── README.md
├── ASGairlinespipeline.ipynb
├── ASGAirlinesdashboard find.pbix
│
├── fact_flights.csv
├── fact_bookings.csv
├── fact_payments.csv
├── dim_route.csv
├── dim_airline.csv
├── dim_date.csv
├── dim_passenger.csv
├── kpi_summary.csv
├── quality_summary.csv
│
├── dashboard_page1.jpg
├── dashboard_page2.jpg
├── dashboard_page3.jpg
├── dashboard_page4.jpg
└── final_summary.jpg

