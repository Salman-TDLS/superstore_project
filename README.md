# Chicago Airbnb Data Analysis using Excel 
-------------------------------------------------------------

This project converts raw InsideAirbnb listings into a **clean, interactive Excel-style dashboard**.  
The workflow maps host hot-spots, review trends, and revenue potential across Chicago’s neighbourhoods.

--------------------------------------------------------------------------------------------------------------------------------
## Steps Involved
### 1. Cleaning Data
- Removed duplicate listing IDs to guarantee uniqueness.  
- Resolved nulls in critical columns (`neighbourhood_cleansed`, `price`, `room_type`).  
- Parsed currency strings to floats and enforced correct data types.  

### 2. Data Analysis
- Built PivotTables to summarise **listing density, review share, and estimated revenue**.  
- Engineered *availability-adjusted revenue* = `price × (365 − availability_365)`.  
- Added slicers (neighbourhood, room type, host status) for one-click drill-downs.  
- Supplemented Excel views with a Python script (`ga_airbnb_analysis.py`) for automated CSV exports and charts.  

### 3. Insights
#### Neighbourhood Activity
- **Top-performing areas**: West Town (636 active listings), Near North Side (561), Lake View (481).  
- **Least-active pockets**: Fuller Park, Riverdale, Burnside each host fewer than 10 listings.  

#### Guest Preference
- **48 %** of all reviews belong to **entire-home rentals**, signalling a clear market tilt away from private-room offerings.  

#### Revenue Outlier
- Highest-earning host is projected to gross **≈ $1.22 M** annually, proving the long-tail upside of standout listings.  

#### Host Growth Over Time
- Active host count rises year-over-year (2014 → 2023) with an average **7 % annual increase**, reflecting a healthy supply pipeline.  

----------------------------------------------------------------
