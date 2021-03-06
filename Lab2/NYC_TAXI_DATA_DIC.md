# NYC Taxi Data  Dictionary

* [Yellow Taxi Trip Records](#Yellow-Taxi-Trip-Records-Data-Dictionary)
* [Green Taxi Trip Records](#Green-Taxi-Trip-Records-Data-Dictionary)
* [FHV Trip Record](#FHV-Trip-Records-Data-Dictionary)



### Yellow Taxi Trip Records Data Dictionary

| Field Name            | Description                                                  |
| :-------------------- | :----------------------------------------------------------- |
| VentorID              | A code indicating the TPEP provider that provided the record.<br/>**1= Creative Mobile Technologies, LLC; 2= VeriFone Inc.** |
| tpep_pickup_datetime  | The date and time when the meter was engaged.                |
| tpep_dropoff_datetime | The date and time when the meter was disengaged.             |
| Passenger_count       | The number of passengers in the vehicle.<br/>This is a driver-entered value. |
| Trip_distance         | The elapsed trip distance in miles reported by the taximeter. |
| PULocationID          | TLC Taxi Zone in which the taximeter was engaged             |
| DOLocationID          | TLC Taxi Zone in which the taximeter was disengaged          |
| RateCodeID            | The final rate code in effect at the end of the trip.<br/>**1= Standard rate<br/>2=JFK<br/>3=Newark<br/>4=Nassau or Westchester<br/>5=Negotiated fare<br/>6=Group ride** |
| Store_and_fwd_flag    | This flag indicates whether the trip record was held in vehicle<br/>memory before sending to the vendor, aka “store and forward,”<br/>because the vehicle did not have a connection to the server.<br/>**Y= store and forward trip<br/>N= not a store and forward trip** |
| Payment_type          | A numeric code signifying how the passenger paid for the trip.<br/>**1= Credit card<br/>2= Cash<br/>3= No charge<br/>4= Dispute<br/>5= Unknown<br/>6= Voided trip** |
| Fare_amount           | The time-and-distance fare calculated by the meter.          |
| Extra                 | Miscellaneous extras and surcharges. Currently, this only includes<br/>the $50 and \$1 rush hour and overnight charges. |
| MTA_tax               | $0.50 MTA tax that is automatically triggered based on the metered<br/>rate in use. |
| Improvement_surcharge | $0.30 improvement surcharge assessed trips at the flag drop. The<br/>improvement surcharge began being levied in 2015. |
| Tip_amount            | Tip amount – This field is automatically populated for credit card<br/>tips. Cash tips are not included. |
| Tolls_amount          | Total amount of all tolls paid in trip.                      |
| Total_amount          | The total amount charged to passengers. Does not include cash tips. |





### Green Taxi Trip Records Data Dictionary

| Field Name            | Description                                                  |
| :-------------------- | :----------------------------------------------------------- |
| VentorID              | A code indicating the TPEP provider that provided the record.<br/>**1= Creative Mobile Technologies, LLC; 2= VeriFone Inc.** |
| lpep_pickup_datetime  | The date and time when the meter was engaged.                |
| lpep_dropoff_datetime | The date and time when the meter was disengaged.             |
| Passenger_count       | The number of passengers in the vehicle.<br/>This is a driver-entered value. |
| Trip_distance         | The elapsed trip distance in miles reported by the taximeter. |
| PULocationID          | TLC Taxi Zone in which the taximeter was engaged             |
| DOLocationID          | TLC Taxi Zone in which the taximeter was disengaged          |
| RateCodeID            | The final rate code in effect at the end of the trip.<br/>**1= Standard rate<br/>2=JFK<br/>3=Newark<br/>4=Nassau or Westchester<br/>5=Negotiated fare<br/>6=Group ride** |
| Store_and_fwd_flag    | This flag indicates whether the trip record was held in vehicle<br/>memory before sending to the vendor, aka “store and forward,”<br/>because the vehicle did not have a connection to the server.<br/>**Y= store and forward trip<br/>N= not a store and forward trip** |
| Payment_type          | A numeric code signifying how the passenger paid for the trip.<br/>**1= Credit card<br/>2= Cash<br/>3= No charge<br/>4= Dispute<br/>5= Unknown<br/>6= Voided trip** |
| Fare_amount           | The time-and-distance fare calculated by the meter.          |
| Extra                 | Miscellaneous extras and surcharges. Currently, this only includes<br/>the $50 and \$1 rush hour and overnight charges. |
| MTA_tax               | $0.50 MTA tax that is automatically triggered based on the metered<br/>rate in use. |
| Improvement_surcharge | $0.30 improvement surcharge assessed trips at the flag drop. The<br/>improvement surcharge began being levied in 2015. |
| Tip_amount            | Tip amount – This field is automatically populated for credit card<br/>tips. Cash tips are not included. |
| Tolls_amount          | Total amount of all tolls paid in trip.                      |
| Total_amount          | The total amount charged to passengers. Does not include cash tips. |
| Trip_type             | A code indicating whether the trip was a street-hail or a dispatch<br/>that is automatically assigned based on the metered rate in use but<br/>can be altered by the driver.<br/>**1= Street-hail<br/>2= Dispatch** |







### FHV Trip Records Data Dictionary

| Field Name           | Description                                                  |
| -------------------- | ------------------------------------------------------------ |
| Dispatching_base_num | The TLC Base License Number of the base that dispatched the trip |
| Pickup_datetime      | The date and time of the trip pick-up                        |
| DropOff_datetime     | The date and time of the trip dropoff                        |
| PULocationID         | TLC Taxi Zone in which the trip began                        |
| DOLocationID         | TLC Taxi Zone in which the trip ended                        |
| SR_Flag              | Indicates if the trip was a part of a shared ride chain offered by a<br/>High Volume FHV company (e.g. Uber Pool, Lyft Line). For shared<br/>trips, the value is 1. For non-shared rides, this field is null.<br/><br/>**NOTE:** For most High Volume FHV companies, only shared rides that<br/>were requested AND matched to another shared-ride request over<br/>the course of the journey are flagged. However, Lyft (base license<br/>numbers **B02510 + B02844**) also flags rides for which a shared ride<br/>was requested but another passenger was not successfully matched<br/>to share the trip—therefore, trips records with SR_Flag=1 from those<br/>two bases could indicate EITHER a first trip in a shared trip chain OR<br/>a trip for which a shared ride was requested but never matched.<br/>Users should anticipate an overcount of successfully shared trips<br/>completed by Lyft. |

