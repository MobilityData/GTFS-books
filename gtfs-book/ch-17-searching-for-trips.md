## 17. Searching for Trips**

This section shows you how to search for trips in a GTFS feed based on
specified times and stops.

The three scenarios covered are:

* Finding all stops departing from a given stop after a certain time
* Finding all stops arriving at a given stop before a certain time
* Finding all trips between two stops after a given time

The first and second scenarios are the simplest, because they only rely
on a single end of each trip. The third scenario is more complex because
you have to ensure that each trip returned visits both the start and
finish stops.

When searching for trips, the first thing you need to know is which
services are running for the given search time.

### Finding Service IDs

The first step in searching for trips is to determine which services are
running. To begin with, you need to find service IDs for a given date.
You then need to handle exceptions accordingly. That is, you need to add
service IDs and remove service IDs based on the rules in
`calendar_dates.txt`.

***Note:** In *Scheduling Past Midnight*, you were shown how
GTFS works with times past midnight. The key takeaway from this is that
you have to search for trips for two sets of service IDs. This is
covered as this chapter progresses.*

In Australia, 27 January 2014 (Monday) was the holiday for Australia
Day. This is used as an example to demonstrate how to retrieve service
IDs.

Firstly, you need the main set of service IDs for the day. The following
SQL query achieves this.

```sql
SELECT service_id FROM calendar
  WHERE start_date <= '20140127' AND end_date >= '20140127'
  AND monday = 1;
 
# Result: 1, 2, 6, 9, 18, 871, 7501
```

Next you need to find service IDs that are to be excluded, as achieved
by the following SQL query.

```sql
SELECT service_id FROM calendar_dates
  WHERE date = '20140127' AND exception_type = 2;

# Result: 1, 2, 6, 9, 18, 871, 7501
```

Finally, you need to find service IDs that are to be added. This query
is identical to the previous query, except for the different
`exception_type` value.

```sql
SELECT service_id FROM calendar_dates
  WHERE date = '20140127' AND exception_type = 1;

# Result: 12, 874, 4303, 7003
```

You can combine these three queries all into a single query in SQLite
using `EXCEPT` and `UNION`, as shown in the following SQL query.

```sql
SELECT service_id FROM calendar
  WHERE start_date <= '20140127' AND end_date >= '20140127'
  AND monday = 1
  
UNION
  
SELECT service_id FROM calendar_dates
  WHERE date = '20140127' AND exception_type = 1
  
EXCEPT
  
SELECT service_id FROM calendar_dates
  WHERE date = '20140127' AND exception_type = 2;

# Result: 12, 874, 4304, 7003
```

Now when you search for trips, only trips that have a matching
`service_id` value are included.

### Finding Trips Departing a Given Stop

In order to determine the list of services above, a base timestamp on
which to search is needed. For the purposes of this example, assume that
timestamp is 27 January 2014 at 1 PM (`13:00:00` when using GTFS).

This example searches for all services departing from Adelaide Railway
Station, which has stop ID `6665` in the Adelaide Metro GTFS feed. To
find all matching stop times, the following query can be performed.

```sql
SELECT * FROM stop_times
  WHERE stop_id = '6665'
  AND departure_time >= '13:00:00'
  AND pickup_type = 0
  ORDER BY departure_time;
```

This returns a series of stop times that match the given criteria. The
only problem is it does not yet take into account valid service IDs.

***Note:** This query may also return stop times that are the final stop
on a trip, which is not useful for somebody trying to find departures.
You may want to modify your database importer to override the final stop
time of each trip so its `pickup_type` has a value of `1` (no pick-up) and
its first stop time so it has a `drop_off_type` of `1` (no drop-off).*

To make sure only the correct trips are returned, join `stop_times`
with `trips` using `trip_id`, and then include the list of service
IDs. For the purposes of this example the service IDs, stop ID and
departure time are being hard-coded. You can either embed a sub-query,
or include the service IDs via code.

```sql
SELECT t.*, st.* FROM stop_times st, trips t
  WHERE st.stop_id = '6665'
  AND st.trip_id = t.trip_id
  AND t.service_id IN ('12', '874', '4304', '7003')
  AND st.departure_time >= '13:00:00'
  AND st.pickup_type = 0
  ORDER BY st.departure_time;
```

This gives you the final list of stop times matching the desired
criteria. You can then decide specifically which data you need to
retrieve; you now have the `trip_id`, meaning you can find all stop
times for a given trip if required.

If you need to restrict the results to only those that occur after the
starting stop, you can retrieve stop times with only a `stop_sequence`
larger than that of the stop time returned in the above query.

### Finding Trips Arriving at a Given Stop

In order to find the trips arriving at a given stop before a specified
time, it is just a matter of making slight modifications to the above
query. Firstly, check the `arrival_time` instead of
`departure_time`. Also, check the `drop_off_type` value instead of
`pickup_type`.

```sql
SELECT t.*, st.* FROM stop_times st, trips t
  WHERE st.stop_id = '6665'
  AND st.trip_id = t.trip_id
  AND t.service_id IN ('12', '874', '4304', '7003')
  AND st.arrival_time <= '13:00:00'
  AND st.drop_off_type = 0
  ORDER BY st.arrival_time DESC;
```

For this particular data set, there are trips from four different routes
returned. If you want to restrict this to a particular route, you can
filter on the `t.route_id` value.

```sql
SELECT t.*, st.* FROM stop_times st, trips t
  WHERE st.stop_id = '6665'
  AND st.trip_id = t.trip_id
  AND t.service_id IN ('12', '874', '4304', '7003')
  AND t.route_id = 'BEL'
  AND st.arrival_time <= '13:00:00'
  AND st.drop_off_type = 0
  ORDER BY st.arrival_time DESC;
```

### Performance of Searching Text-Based Times

These examples search using text-based arrival/departure times (such as
`13:00:00`). This works because the GTFS specification mandates that
all times are `HH:MM:SS` format (although `H:MM:SS` is allowed for times
earlier than 10 AM).

Doing this kind of comparison (especially if you are scanning millions
of rows) is quite slow and expensive. It is more efficient to convert
all times stored in the database to integers that represent the number
of seconds since midnight.

***Note:** The GTFS specification states that arrival and departure times
are "noon minus 12 hours" in order to account for daylight savings time.
This is effectively midnight, except for the days that daylight savings
starts or finishes.*

In order to achieve this, you can convert the text-based time to an
integer with `H * 3600 + M * 60 + S`. For example, `13:35:21` can
be converted using the following steps.

```
  (13 * 3600) + (35 * 60) + (21)
= 46800 + 2100 + 21
= 48921
```

You can then convert back to hours, minutes and seconds in order to
generate timestamps in your application as shown in the following
algorithm.

```
H = floor( 48921 / 3600 )
  = floor( 13.59 )
  = 13
  
M = floor( 48921 / 60 ) % 60
  = floor( 815.35 ) % 60
  = 815 % 60
  = 35
  
S = 48921 % 60
  = 21
```

### Finding Trips Between Two Stops

Now that you know how to look up a trip from or to a given stop, the
previous query can be expanded so both the start and finish stop are
specified. The following example finds trips that depart after 1 PM. The
search only returns trips departing from *Adelaide Railway Station*
(stop ID `6665`) are shown. Additionally, only trips that then visit
*Blackwood Railway Station* (stop IDs `6670` and `101484`) are
included.

In order to achieve this, the following changes must be made to the
previous examples.

* **Join against stop_times twice.** Once for the departure stop time
  and once for the arrival stop time.
* **Allow for multiple stop IDs at one end.** The destination in this
  example has two platforms, so you need to check both of them.
* **Ensure the departure time is earlier than the arrival time.
  **Otherwise trips heading in the opposite direction may also be
  returned.

The following query demonstrates how this is achieved.

```sql
SELECT t.*, st1.*, st2.*
  FROM trips t, stop_times st1, stop_times st2
  WHERE st1.trip_id = t.trip_id
  AND st2.trip_id = t.trip_id
  AND st1.stop_id = '6665'
  AND st2.stop_id IN ('6670', '101484')
  AND t.service_id IN ('12', '874', '4304', '7003')
  AND st1.departure_time >= '13:00:00'
  AND st1.pickup_type = 0
  AND st2.drop_off_type = 0
  AND st1.departure_time < st2.arrival_time
  ORDER BY st1.departure_time;
```

In this example, the table alias `st1` is used for the departure stop
time. Once again, the stop ID must match, as well as the departure time
and pick-up type.

For the arrival stop time the alias `st2` is used. This table also
joins the `trips` table using `trip_id`. Since the destination has
multiple stop IDs, the SQL `IN` construct is used. The arrival time is
not important in this example, so only the departure is checked.

The final thing to check is that the departure occurs before the
arrival. If you do not perform this step, then trips traveling in the
opposite direction may also be returned.

***Note:** Technically, there may be multiple results returned for the
same trip. For some transit agencies, a single trip may visit the a stop
more than once. If this is the case, you should also check the trip
duration (arrival time minus departure time) and use the shortest trip
when the same trip is returned multiple times.*

### Accounting for Midnight

As discussed previously, GTFS has the ability for the trips to depart or
arrive after midnight for a given service day without having to specify
it as part of the next service day. Consequently, while the queries
above are correct, they do not necessarily paint the full picture.

In reality, when performing a trip search, you need to take into account
trips that have wrapped times (for instance, where 12:30 AM is specified
as `24:30:00`). If you want to find trips that depart after 12:30 AM
on a given day, you need to check for trips departing after
`00:30:00` on that day, as well as for trips departing at
`24:30:00` on the previous day.

This means that for each trip search you are left with two sets of
trips, which you must then merge and present as appropriate.

***Note:** In reality, agencies generally do not have trips that overlap
from multiple service days, so technically you often only need one query
(for example, a train service might end on 12:30 AM then restart on the
next service day at 4:30 AM). If your app / web site only uses a single
feed where you can tune your queries manually based on how the agency
operates, then you can get away with only querying a single service day.
On the other hand, if you are building a scalable system that works with
data from many agencies, then you need to check both days.*

To demonstrate how this works in practice, the following example
searches for all trips that depart after 12:30:00 AM on 14 March 2014.
The examples earlier in this chapter showed how to find the service IDs
for a given date. To account for midnight, service IDs for both March 14
(the "main" service date) and March 13 (the overlapping date) need to be
determined.

Assume that March 13 has a service ID of `C1` and March 14 has a
service ID of `C2`. First you need to find the departures for March
14, as shown in the following query.

```sql
SELECT t.*, st.* FROM stop_times st, trips t
  WHERE st.stop_id = 'S1'
  AND st.trip_id = t.trip_id
  AND t.service_id IN ('C1')
  AND st.departure_time >= '00:30:00'
  AND st.pickup_type = 0
  ORDER BY st.departure_time;
```

The resultant list needs to be combined with the trips that depart after
midnight from the March 13 service. To check this, it is just a matter
of swapping in the right service IDs, then adding 24 hours to the search
time.

```sql
SELECT t.*, st.* FROM stop_times st, trips t
  WHERE st.stop_id = 'S1'
  AND st.trip_id = t.trip_id
  AND t.service_id IN ('C2')
  AND st.departure_time >= '24:30:00'
  AND st.pickup_type = 0
  ORDER BY st.departure_time;
```

In order to get your final list of trips you must combine the results
from both of these queries. If you are generating complete timestamps in
order to present the options to your users, just remember to account for
the results from the second query being 24 hours later.

