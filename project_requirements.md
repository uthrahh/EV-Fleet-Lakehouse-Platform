An **enterprise-style data Lakehouse for a commercial EV fleet operator** managing thousands of connected delivery vehicles. The platform processes high-volume vehicle telemetry—including GPS location, speed, battery state-of-charge, trip activity, charging events, vehicle health, fault codes, and maintenance records—through **Bronze, Silver, and Gold layers** using Databricks, PySpark, and Delta Lake.

The platform supports **incremental ingestion, scalable distributed processing, data quality validation, governance, and analytics-ready data modeling**. Curated Gold datasets are consumed by Power BI dashboards to analyze **fleet utilization, battery efficiency, charging performance, vehicle health, maintenance requirements, safety, and overall operational performance**.

**Industry / Domain** → Connected Mobility & Commercial EV Fleet Telematics

**Enterprise / Company** → A large-scale commercial EV fleet operator managing thousands of connected delivery vehicles and using vehicle telemetry to optimize fleet utilization, battery and charging performance, vehicle health, preventive maintenance, safety, and operational efficiency.

- **Tech stack**
    
    Lakehouse Platform → Databricks Free Edition
    
    Storage
    → Databricks-managed storage
    → Unity Catalog Volumes
    
    Distributed Processing → Apache Spark / PySpark
    
    Storage/Table Format → Delta Lake
    
    Ingestion → Databricks Auto Loader
    
    Streaming → Spark Structured Streaming
    
    SQL → Databricks SQL
    
    Architecture → Medallion Architecture
    
    Governance → Unity Catalog
    
    Analytics Engineering → dbt
    
    Data Quality → dbt Tests + PySpark validation
    
    Orchestration → Apache Airflow
    
    Languages → Python + SQL
    
    Configuration → YAML + environment variables
    
    Monitoring → Databricks logs + Delta audit tables
    
    Development → Databricks Notebooks + VS Code
    
- **Requirements**
    
    ## Business Problem
    
    > **Why does the EV fleet company need this Lakehouse?**
    > 
    
    The fleet operator lacks a centralized, reliable analytics platform for combining high-volume vehicle telemetry, charging, trip, diagnostic, and maintenance data, making it difficult to monitor fleet performance, identify inefficient vehicles, detect battery/vehicle-health issues, optimize charging, and make timely operational decisions.
    
    | Business area | Problem |
    | --- | --- |
    | Fleet utilization | Which vehicles are being underutilized or overutilized? |
    | Battery | Which vehicles have abnormal battery consumption or declining efficiency? |
    | Charging | Where are charging delays, excessive charging times, or inefficient charging patterns occurring? |
    | Operations | How much distance/trip activity is each vehicle completing? |
    | Vehicle health | Which vehicles repeatedly generate diagnostic faults? |
    | Maintenance | Which vehicles are due or overdue for maintenance? |
    | Safety | Which vehicles/drivers show harsh braking, speeding, etc.? |
    | Reliability | Which vehicles experience the most failures/downtime? |
    | Geography | How does fleet performance differ by city/depot/region? |
    | Management | What are the overall fleet KPIs and trends? |
    
    ---
    
    ## Source Systems
    
    > **Where would the data required to solve those problems originate in a real EV company?**
    > 
    
    1. **Vehicle telematics / IoT System** - Connected vehicles continuously transmit telemetry.
    
    vehicle_id
    timestamp
    latitude
    longitude
    speed_kmph
    odometer_km
    battery_soc
    battery_voltage
    battery_temperature
    motor_temperature
    ignition_status
    
    2. **Fleet management system** - Contains relatively stable information about the vehicles.
    
    vehicle_id
    registration_number
    vehicle_model
    manufacturer
    battery_capacity_kwh
    vehicle_type
    purchase_date
    commission_date
    fleet_id
    depot_id
    status
    
    3. **Trip management system -** tracks completed deleivery trips.
    
    trip_id
    vehicle_id
    driver_id
    start_timestamp
    end_timestamp
    start_location
    end_location
    distance_km
    trip_duration_minutes
    energy_consumed_kwh
    
    4. **Charging management system -** Charging stations or charging software generate charging-session data.
    
    charging_session_id
    vehicle_id
    charger_id
    station_id
    start_timestamp
    end_timestamp
    soc_start
    soc_end
    energy_delivered_kwh
    charging_duration_minutes
    cost
    
    5. **Vehicle Diagnostics System** - The vehicle/BMS/diagnostics system generates faults.
    
    fault_event_id
    vehicle_id
    timestamp
    fault_code
    fault_category
    severity
    component
    description
    
    6. **Maintenance management system** - Contains workshop/service records.
    
    maintenance_id
    vehicle_id
    service_date
    maintenance_type
    component
    odometer_km
    cost
    downtime_hours
    next_service_date
    
    ---
    
    ## Business Entities
    
    > **What are the major "things" the business needs to keep track of?**
    > 
    
    | Entity | Meaning |
    | --- | --- |
    | **Vehicle** | Individual commercial EV |
    | **Vehicle Model** | Technical specification/model of vehicle |
    | **Driver** | Driver assigned to vehicles/trips |
    | **Fleet** | Logical group of vehicles |
    | **Depot** | Operational base for vehicles |
    | **Location** | Geographic information |
    | **Telemetry Event** | Vehicle sensor snapshot at a particular time |
    | **Trip** | Vehicle journey/delivery operation |
    | **Charging Session** | One vehicle charging event |
    | **Charging Station** | Physical charging location |
    | **Charger** | Individual charging unit |
    | **Fault Event** | Diagnostic issue generated by a vehicle |
    | **Maintenance Event** | Service/repair activity |
    | **Date/Time** | Analytical time dimension |
    
    ```
                       VEHICLE
                          │
          ┌───────────────┼─────────────────┐
          │               │                 │
          ▼               ▼                 ▼
     TELEMETRY           TRIP        CHARGING SESSION
          │               │                 │
          │               ▼                 ▼
          │             DRIVER           CHARGER
          │                                  │
          ▼                                  ▼
     FAULT EVENT                    CHARGING STATION
          │
          ▼
    MAINTENANCE EVENT
    ```
    
    ---
    
    ## Analytics Requirements
    
    > **What must the platform allow the business to analyze?**
    > 
    1. **Fleet Utilization Analytics**
        
        The platform should allow operations teams to:
        
        - Monitor active vs inactive vehicles.
        - Measure distance travelled by vehicle/fleet.
        - Measure trips completed.
        - Measure operating vs idle time.
        - Identify underutilized vehicles.
        - Compare utilization across depots, cities, fleets and vehicle models.
        - Analyze utilization trends over time.
        
        Required entities: Vehicle + Trip + Telemetry + Depot + Date.
        
    2. **Battery & Energy Analytics**
        
        The platform should allow teams to:
        
        - Monitor battery State of Charge (SOC).
        - Measure energy consumed.
        - Calculate energy efficiency.
        - Compare efficiency across vehicles/models.
        - Identify abnormal battery consumption.
        - Monitor battery temperature.
        - Identify vehicles showing possible battery degradation.
        - Analyze battery performance over time.
        
        Required entities: Vehicle + Telemetry + Trip + Date.
        
    3. **Charging Analytics**
        
        The platform should allow teams to:
        
        - Monitor charging sessions.
        - Measure charging duration.
        - Measure energy delivered.
        - Calculate SOC gained per session.
        - Analyze charger/station utilization.
        - Identify unusually slow charging sessions.
        - Identify charging failures where available.
        - Compare charging behaviour across vehicles and stations.
        
        Required entities: Vehicle + Charging Session + Charger + Station + Date.
        
    4. **Vehicle Health Analytics**
        
        The platform should allow teams to:
        
        - Monitor fault events.
        - Analyze fault severity.
        - Identify vehicles generating repeated faults.
        - Identify frequently failing components.
        - correlate faults with telemetry such as battery/motor temperature.
        - Track vehicle-health trends.
        - Identify vehicles requiring inspection.
        
        Required entities: Vehicle + Telemetry + Fault Event + Date.
        
    5. **Maintenance & Reliability Analytics**
        
        The platform should allow teams to:
        
        - Monitor scheduled maintenance.
        - Identify overdue maintenance.
        - Analyze maintenance frequency.
        - Measure vehicle downtime.
        - Measure maintenance cost.
        - Identify frequently repaired vehicles.
        - Analyze failures before maintenance events.
        - Compare reliability across vehicle models.
        
        Required entities: Vehicle + Maintenance Event + Fault Event + Date.
        
    6. **Safety & Operations Analytics**
        
        Depending on the telemetry available:
        
        - Detect speeding events.
        - Detect harsh acceleration.
        - Detect harsh braking.
        - Monitor driving behavior.
        - Compare safety indicators by driver.
        - Compare performance geographically.
        - Identify operational anomalies.
        
        Required entities: Vehicle + Driver + Trip + Telemetry + Location + Date.
        
    
    ---
    
    ## KPIs
    
    ### Fleet
    
    | KPI | Meaning |
    | --- | --- |
    | Total Vehicles | Number of vehicles in fleet |
    | Active Vehicles | Vehicles operational during selected period |
    | Fleet Utilization % | Percentage of available fleet being utilized |
    | Total Trips | Number of completed trips |
    | Total Distance | Distance travelled |
    | Avg Distance / Vehicle | Average distance travelled per vehicle |
    | Avg Trips / Vehicle | Average trips completed per vehicle |
    | Idle Time | Time vehicles remain inactive while operational |
    
    ### Battery & Energy
    
    | KPI | Meaning |
    | --- | --- |
    | Average SOC | Average battery State of Charge |
    | Energy Consumed | Total kWh consumed |
    | Energy Efficiency | Distance travelled per kWh |
    | Avg Battery Temperature | Fleet battery temperature |
    | Low-SOC Events | Number of critically low battery events |
    | High-Temperature Events | Abnormal battery-temperature events |
    
    ### Charging
    
    | KPI | Meaning |
    | --- | --- |
    | Total Charging Sessions | Number of sessions |
    | Energy Delivered | Total charging energy |
    | Avg Charging Duration | Average session duration |
    | Avg SOC Gain | SOC increase per charging session |
    | Charger Utilization % | Usage of available charging capacity |
    | Avg Energy / Session | Energy delivered per charging event |
    
    ### Vehicle Health
    
    | KPI | Meaning |
    | --- | --- |
    | Total Charging Sessions | Number of sessions |
    | Energy Delivered | Total charging energy |
    | Avg Charging Duration | Average session duration |
    | Avg SOC Gain | SOC increase per charging session |
    | Charger Utilization % | Usage of available charging capacity |
    | Avg Energy / Session | Energy delivered per charging event |
    
    ### Maintenance
    
    | KPI | Meaning |
    | --- | --- |
    | Maintenance Events | Number of services/repairs |
    | Maintenance Cost | Total maintenance expenditure |
    | Avg Maintenance Cost / Vehicle | Fleet maintenance cost normalized by vehicle |
    | Downtime Hours | Vehicle time unavailable |
    | Overdue Maintenance | Vehicles past scheduled service |
    | Mean Time Between Failures | Average operating time between failures |
    
    ### Safety
    
    | KPI | Meaning |
    | --- | --- |
    | Speeding Events | Speed threshold violations |
    | Harsh Braking Events | Sudden braking |
    | Harsh Acceleration Events | Sudden acceleration |
    | Safety Events / 100 km | Normalized safety-event rate |

---

## Visualizations

### Fleet Executive Overview

> **How is the fleet performing right now/over the selected period?**
> 

KPI Cards:

- Total Vehicles
- Active Vehicles
- Fleet Utilization %
- Total Trips
- Total Distance
- Energy Efficiency
- Critical Faults
- Downtime

Visuals:

- Fleet Utilization trend
- Fleet Status
- Performance by city/depot
- Top vehicles requiring attention

### Fleet Utilization

> **Are we using our vehicles effectively?**
> 
- Fleet Utilization %
- Active Vehicles
- Trips
- Distance
- Idle time
- Utilization trend
- Vehicle utilization ranking
- Utilization by depot
- Utilization by model
- Underutilized vehicles

### Battery & Energy

> **Are our EV batteries operating efficiently?**
> 
- Energy Efficiency
- Average SOC
- Energy Consumed
- Low-SOC Events
- Battery temperature
- Energy efficiency trend
- Efficiency by vehicle
- Efficiency by vehicle model
- SOC distribution
- Vehicles with abnormal energy consumption
- Battery-temperature anomalies

### Charging Performance

> **Is the fleet charging efficiently?**
> 
- Charging Sessions
- Energy Delivered
- Average Charging Duration
- Average SOC Gain
- Charger Utilization
- Charging sessions over time
- Charging duration distribution
- Station utilization
- Energy delivered by station
- Vehicles with unusually long charging sessions

### Vehicle Health & Maintenance

- Total Faults
- Critical Faults
- Vehicles with faults
- Maintenance Cost
- Downtime
- Overdue Maintenance
- Fault trend
- Faults by component
- Faults by severity
- Maintenance cost by vehicle
- Downtime by vehicle
- Most problematic vehicles

### Safety & Operations

- Speeding events
- Harsh braking
- Harsh acceleration
- Safety events / 100 km
- Safety trend
- Events by driver
- Events by vehicle
- Events by location
- Driver safety ranking

## Problem Statement

A large commercial electric-vehicle fleet generates high-volume operational data across vehicle telemetry, trips, charging systems, diagnostics, and maintenance platforms. Because these datasets originate from independent systems and differ in structure, frequency, and quality, fleet operators face difficulty obtaining a unified and reliable view of vehicle utilization, battery efficiency, charging performance, vehicle health, maintenance requirements, safety, and overall fleet operations.

This project builds an enterprise-style data Lakehouse that incrementally ingests and integrates connected-vehicle data into a governed Bronze, Silver, and Gold architecture. Raw operational data is cleaned, standardized, validated, and transformed into analytics-ready datasets that enable reliable fleet KPIs and Power BI dashboards for operational monitoring and decision-making.

## Success Criteria

**Data ingestion -** New source data can be processed incrementally without reprocessing the entire historical dataset.

**Data integrity -** Reprocessing the same batch doesn't create duplicate business records.

**Data quality** - Automated rules detect invalid data.

**Schema handling -** Expected schema changes can be handled deliberately while incompatible changes are detected.

**Delta Lake -** Successfully demonstrate:

- ACID writes
- `MERGE`
- Schema enforcement
- Schema evolution
- Time travel
- Table history
- `OPTIMIZE`
- `VACUUM` where supported/appropriate

**Medallion architecture** - Clear separation exists between:

Bronze: Raw / source-aligned
        ↓
Silver: Cleaned, Validated, Deduplicated, Standardized, Integrated
        ↓
Gold: Business-level, Analytics-ready, Aggregated, BI optimized

**Data quality target -** 

For example: **100% of Gold records must pass defined critical data-quality rules before publication.**

That doesn't mean source data magically has 100% quality.

It means bad records don't reach Gold unnoticed.

**Referential integrity**

**Orchestration -** 

Airflow can execute: Ingestion → Bronze → Bronze validation → Silver → Silver validation → dbt / Gold → Gold validation → Complete

with: dependencies, retries, failure detection, logging, rerun capability.

**Governance -** 

Unity Catalog should demonstrate:

- Catalog/schema/table organization
- Ownership
- Permissions
- Governed access
- Metadata
- Lineage where supported

**Auditability -** Every pipeline execution should produce something like:

run_id
pipeline_name
start_time
end_time
status

source_records
bronze_records
silver_records
gold_records

inserted_records
updated_records
rejected_records

quality_status

stored in Delta audit tables.

**Analytics -** Gold tables support the defined Power BI KPIs without Power BI having to perform major data-cleaning logic.