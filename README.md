# uber-trip-analysis-powerbi


This project presents an interactive dashboard designed to analyze Uber trip data, revealing key trends and insights such as ride demand patterns, peak hours, location-based activity, payment preferences, and revenue generation. It is built to support data-driven decision-making for urban transport analysis.

---

## 📊 Dashboard Structure

### 1️⃣ Overview Analysis
- KPIs
- payment type analysis
- Day/Night analysis
- vehicle type analysis
- Total month analysis by each day
- Location Analysis

### 2️⃣ Time Analysis
- 24 hrs trips analysis
- bookings by days(Mon, Tue, Web, Thurs, Fri,Sat,Sun) analysis
- pickup time analysis

### 3️⃣ Details
Deep-dive analysis for granular insights:
- Trip-level data exploration with filters
- Custom filters by location, time, payment type, and vehicle


---

## 🧾 Dataset Description

###  `Trip Details`
Contains detailed records of individual Uber rides:

| Column            | Description |
|------------------|-------------|
| `Trip ID`         | Unique trip identifier |
| `Pickup Time`     | Ride start time |
| `Drop Off Time`   | Ride end time |
| `Passenger Count` | Number of passengers |
| `Trip Distance`   | Distance of the trip in miles |
| `PULocationID`    | Pickup location ID |
| `DOLocationID`    | Drop-off location ID |
| `Payment Type`    | Mode of payment (Credit, Cash, Wallet, etc.) |
| `Fare Amount`     | Base fare charged |
| `Surge Fee`       | Additional fee during high demand |
| `Vehicle`         | Uber service type (UberX, UberXL, etc.) |

###  `Location Table`
Maps numeric location IDs to area names and cities.

| Column       | Description |
|--------------|-------------|
| `LocationID` | Unique location identifier |
| `Location`   | Area or neighborhood name |
| `City`       | Corresponding city |


###  `Calendar Table`
A custom calendar table to support time-based filtering and aggregation.

| Column     | Description |
|------------|-------------|
| `Date`     | Full date (YYYY-MM-DD) |
| `Day Num`  | Numeric value of the weekday (1=Sunday, 7=Saturday) |
| `Day Name` | Name of the day (e.g., Monday, Tuesday) |

---

## 🛠️ Tools & Technologies

- **Power BI** (or Tableau) for dashboard development
- **Excel/CSV** for raw data storage
- **DAX / Calculated Columns** for measures and filters
- **Git & GitHub** for version control

---
### DAX formulas used for KPIs and some measures

Avg Bookings amount = DIVIDE([Total Bookings value],[Total Bookings],BLANK())



Avg Trip distance = 
var avgmiles = ROUND(AVERAGE('Trip Details'[trip_distance]),0)
return 
CONCATENATE(avgmiles, " miles")



Avg Trip Time = 
 var avgtriptime = AVERAGEX('Trip Details',DATEDIFF('Trip Details'[Pickup Time],'Trip Details'[Drop Off Time],MINUTE))
 RETURN
 CONCATENATE(FORMAT(avgtriptime,"0")," min")



Total Bookings = COUNT(('Trip Details'[Trip ID]))



Total Bookings value = SUM('Trip Details'[fare_amount])+SUM('Trip Details'[Surge Fee])



Total Trip distance = 
var totalmiles = SUM('Trip Details'[trip_distance])/1000
return
CONCATENATE(FORMAT(totalmiles,"0")," Kmiles")

Total Trip Distance measure = SUM('Trip Details'[trip_distance])




Farthest Trip = 
VAR MaxDistance =MAX( 'Trip Details'[trip_distance])

VAR PickupLocation =
    LOOKUPVALUE(
        'Location Table'[Location],'Location Table'[LocationID],
        CALCULATE(
            SELECTEDVALUE('Trip Details'[PULocationID]),
            'Trip Details' [trip_distance] = MaxDistance
        )
    )



VAR DropoffLocation =
    LOOKUPVALUE('Location Table'[Location],'Location Table'[LocationID],
        CALCULATE (
            SELECTEDVALUE('Trip Details'[PULocationID]),
            'Trip Details' [trip_distance] = MaxDistance
        )
    )

RETURN
    "Pickup: " & PickupLocation &" -> Drop-off: "& DropoffLocation & " (" & FORMAT(MaxDistance,"0.0") &" miles)"



Most frequent Dropoff point = 
VAR DropOffCounts =
    ADDCOLUMNS(
        SUMMARIZE(
            'Trip Details' ,
            'Location Table' [Location]
        ),
    "DropOffCount" ,
    CALCULATE(
        COUNT('Trip Details' [Trip ID]),
        USERELATIONSHIP( 'Trip Details'[DOLocationID], 'Location Table' [LocationID])
    )
)
VAR RankedDropoffs =
    ADDCOLUMNS(
        DropOffCounts ,
        "Rank" ,
        RANKX(DropOffCounts,[DropOffCount],,DESC,Dense)
    )
VAR TopDropoff=
    FILTER(RankedDropoffs, [Rank] = 1)

RETURN CONCATENATEX(TopDropoff,'Location Table'[Location],",")




Most Frequent pickup point = 

var pickpoint = TOPN(1, 
                       SUMMARIZE('Trip Details','Location Table'[Location],"pickup point",COUNT('Trip Details'[Trip ID])),

                       [pickup point],DESC

)

RETURN CONCATENATEX(pickpoint,'Location Table'[Location],",")

